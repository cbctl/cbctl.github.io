---
layout: post
permalink: /blog/baremetal-networking
title: "DHCP, NAT, VLAN, oh my!"
date: "May 18, 2022"
---

#### Or; some networking stuff I learned for work

At work I am part of a team which builds [liquid metal](https://github.com/weaveworks-liquidmetal?view_as=public),
a set of components to provision Kubernetes on bare-metal. A few months ago we started demoing what we had
so far, and I picked up the ticket to prep the environment, make it reproducible
with some terraform, write all the docs, etc. Because I spent so much time in there
I ended up being the one to then do the thing live a whole bunch of times.
You can see one of these runs at a Cluster API Office Hour [here](https://www.youtube.com/watch?v=3gLPbUG9wwc&ab_channel=MichaelMcCune)
from around the 12 min mark.

I didn't really know anything about networking stuff before this, and therefore had
a fun few days rolling from one problem to the next, learning as I went to build something
which "works well enough" and "is not a security disaster".
I am therefore not presenting my eventual setup as in any way production ready.

Instead I am going to lay out each problem that I had, in turn, and which tool I chose
to solve it. I wont go into deep detail, you can google things once you are sure that
something I mention will solve your particular problem.

This is honestly for my benefit more that anyone else's, since I know
I will forget it all. Annoyingly, because I am useless, I waited a whole 3 months
before writing this, so I have already forgotten a tedious amount. Namely I cannot
remember clearly the things which did not work, the u-turns in the plan I made (there
were a lot of those) and other tangential things I learned but did not end up using.

Maybe it will come back more as I write ¯\\_(ツ)\_/¯ .

So here is what I was trying to achieve/the problems I had to solve:
1. I needed a reproducible bare-metal environment
1. In which I could deploy kubernetes nodes on [MicroVMs][flintlock]
1. Each MicroVM needed to be assigned an IP address
1. But I didn't want to waste precious IPv4 address space, or have the things hosting my kubernetes nodes just sitting there all exposed, so the IPs had to be in a private network
1. And each node should be able to reach the internet
1. Also I had to bare in mind our software's own networking opinions
1. And I didn't want to expose our [MicroVM service][flintlock] to the internet (with or without auth)
1. Yet everything should be reachable from wherever I was controlling the kubernetes clusters

### Equinix

**Problem**: I needed some "bare-metal" (basically some actual hardware and not a VM).
Since anyone should be able to get my demo env running without actually buying machinery,
I needed to get this metal from a service of some sort.

So I used [Equinix Metal][equinix] since we already had an account there.
Like EC2 on AWS, Equinix let me say "I want 2 devices of this type in this location". Unlike
AWS, because the hardware is limited, Equinix could sometimes come back with "Amsterdam
is busy right now", so that was something I had to think about.

(The device type was fairly important as I needed the devices which would end up running
a service called [`flintlock`][flintlock] to have a spare disk which I could then format for
[direct-lvm][directlvm].
But this is a networking essay so I not getting into liquid metal components.)

Another thing I had to think about was how any networking I did had to reckon with,
or indeed be built upon, the options which Equinix was happy to let you tinker with.

So a lot of my subsequent problems were framed around that particular flavour,
and a lot of things I may have chosen to do were dropped simply because Equinix did
not support it.

I created 3 devices and chose 1 to act as my "network control hub" thingy, and the other 2
would be provisioned to run `flintlock`. (I could have made my "hub" share another device,
as I wasn't doing anything so heavy that it would need a lot of space, but I liked
to keep it dedicated for sanity's sake.)

### VLAN

**Problem**: Each MicroVM had to run in a private network so we didn't waste public IPs and for extra security.

I created a [Virtual Local Area Network (VLAN)][vlan] with a private subnet which would
contain all my future MicroVMs.

Setting up a VLAN requires fiddling about with the physical router, which of course
I could not access. So I asked Equinix to make me one.

To get any future MicroVMs inside the network, I first had to put my bare-metal devices
in it. To attach a device to a VLAN, Equinix makes you choose a new networking mode
(again this is because with VLAN setup you have to create interfaces on the router,
and also because Equinix has their own opinions and deliberate limitations).

The options are [Hybrid Bonded][bonded], [Hybrid Unbonded][unbonded], and [Pure Layer 2][l2].
I went with [Hybrid Bonded][bonded] because a) I couldn't get unbonded to work (the docs were
not especially accurate), b) pure L2 is not what I wanted, and c) the bonded setup made
made NAT forwarding rules very easy to write (see next section).

I ran this on each device to attach it to the VLAN:

```bash
# check the module is loaded
modprobe 8021q

# enable the module
echo "8021q" >> /etc/modules-load.d/networking.conf

# add a new interface for the vlan, using the id chosen at creation
ip link add link bond0 name "bond0.$VLAN_ID" type vlan id "$VLAN_ID"

# add an address to the interface. note the $ADDR variable: each device
# added will need its own
# the cidr size i chose somewhat randomly. i don't need tons of space for demos,
# but i wanted enough room to be careless
ip addr add "192.168.1.$ADDR/25" dev "bond0.$VLAN_ID"

# bring up the interface
ip -d link set dev "bond0.$VLAN_ID" up
```

### NAT

**Problem**: The MicroVMs needed to access the internet to download things (like pod images).

Now in their private network, the MicroVMs had no internet.

So I set up [Network Address Translation (NAT)][nat] on my "network hub" device
to forward any traffic come from _inside_ the VLAN through the parent network (via
the bond), but did not open up any traffic coming the other way.

```bash
apt update

# i chose firewalld for this, but there are other things like iptables, ufw, etc
apt install -y firewalld

# enable ip forwarding
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf

# save the settings
sysctl -p

# add a rule to forward any traffic from the internal VLAN interface
# to the external parent interface
firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i "bond0.$VLAN_ID" -o bond0 -j ACCEPT

# add a rule to ensure that any return packets are sent back to the correct
# internal address which made the OG call
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -o bond0 -j MASQUERADE

# restart
firewall-cmd --reload
```

### DHCP

**Problem**: Each MicroVM created by [`flintlock`][flintlock] needed to be automatically assigned an ip address.

For that I used Dynamic Host Configuration Protocol (DHCP). DHCP lets you configure
a pool of addresses, and it will "lease" an IP to whoever comes and asks for one.

So on my "network hub" device I did this:

```bash
apt update

# install a basic server
apt install -y isc-dhcp-server

# create some config
cat <<EOF >/etc/dhcp/dhcpd.conf
default-lease-time 600;
max-lease-time 7200;
authoritative;

# here the subnet, range and router settings were from my VLAN
subnet 192.168.1.0 netmask 255.255.255.128 {
# i used the upper end of my range to avoid clashes with the nodes which
# have the first few ips. obviously this is not an irl situation, but I only
# needed like 10 ips for my demos
  range 192.168.1.26 192.168.1.126;
  option routers 192.168.1.2;
}
EOF

# edit another config file to set it so the server watches for things coming up
# in the VLAN
sed -i "s/INTERFACESv4.*/INTERFACESv4=\"bond0.$VLAN_ID\"/g" /etc/default/isc-dhcp-server

# restart the service
systemctl restart isc-dhcp-server.service
```

### DNS

For a minute there my MicroVMs just would not resolve (connect to stuff on the internet by name rather than by address)
and it was because the `/etc/resolv.conf` files were a bit messed up.

I solved this by using DHCP's `domain-name-servers` option, which meant that when
my DHCP server assigned an IP to a new MicroVM, it would also give it some DNS addresses
which it could use when doing a lookup.

I chose the upstream addresses used by the host on Equinix (found with `resolvectl status`)
and updated my `/etc/dhcp/dhclient.conf`:

```bash
...

subnet 192.168.1.0 netmask 255.255.255.128 {
  range 192.168.1.26 192.168.1.126;
  option routers 192.168.1.2;

  # this line here
  option domain-name-servers 147.75.207.207, 147.75.207.208;
}

...
```

### Flintlock

**Problem**: Don't forget about `flintlock`'s own networking jazz, don't bind it to a public IP

When `flintlock` starts, you bind the service to an address. When I started prepping the
demo env we didn't have any sort of auth in place, so I didn't really want it reachable
from anywhere. With the MicroVM host devices on the VLAN, I could start the services on the
private addresses.

`flintlock` creates a [`macvtap`](https://virt.kernelnewbies.org/MacVTap) device for each MicroVM.
On start, we tell `flintlock` which parent device each `macvtap` device should be
interfacing to.
Without the VLAN, the parent device would be `bond0`. With the VLAN is it now the VLAN
interface, `bond0.$VLAN_ID`. (_Okay, technically with this setup it could be bound
to `bond0` and be fine, but if binding the service to the public address, then the
parent interface_ has _to be `bond0.$VLAN_ID` otherwise the MicroVM will not receive
a public IP. I am still digging into why... watch this space._)

```bash
# in reality i used a systemd service file and a config file,
# but this makes it a little more clear
flintlock run --grpc-endpoint "192.168.1.$ADDR:9090" --parent-iface "bond0.$VLAN_ID"
```

### Tailscale

**Problem**: I needed to be able to reach my `flintlock` servers and kubernetes nodes from my management workstation.

The last piece was now getting into my private network to do stuff. My first thought
was a Virtual Private Network (VPN), but after digging around it just seemed like a lot of work to get that set
up; even with automation it made the "throwaway" environment I wanted a bit more effort.

Then a colleague I asked to sanity check everything I had so far suggested [Tailscale][tailscale]
and omigoodness I love this thing. Seriously when you are done with this crappy blog
post, go read _theirs_. Really fascinating stuff.

Anyway, I realised that what I wanted could be easily achieved with a VPN/[subnet router][router]
(previously a relay node) combo. Basically I could tell the Tailscale Magic to route traffic
from my VLAN subnet to my Tailscale VPN network, which I was connected to on my
personal workstation, far away from my bare-metal devices.

All I had to do was run the following on my "network hub" device.

```bash
curl -fsSL https://tailscale.com/install.sh | sh

tailscale up --advertise-routes=192.168.1.0/25 --authkey "$AUTH_KEY"
```

And this on my local computer:

```bash
tailscale up --accept-routes
```

And that's it. How cool is that?

I know that everything else in this post is like "and I just ran this" and it sounds
super pro like I just magically knew exactly what to do, but in reality those
are summaries of hours fiddling about and tons (TONS!!) of fuckups, mind-changes and backtracking. The Tailscale section
is the only _genuine, effortless_ "just run this" bit. That is how good it is. :chefs_kiss:

&nbsp;

---------------------------------------------

&nbsp;

### Wrap up

The resulting Terraform can be found [here](https://github.com/weaveworks-liquidmetal/getting-started/blob/main/terraform/main.tf).
I had also never really written terraform before, I am sure it shows.

I wish I could provide a list of resources or tutorials which I used to put this all
together but honestly the answer is "all the internet", _all of it_. Aside from a couple of
check-in chats with 2 colleagues, it was just me, a search engine and a terminal.
It was really fun, so I honestly just suggest going for it and trying everything.

I found that googling for networking questions is very different than for googling
literally anything else in software. It felt like every answer was for a very specific
problem, only 2% of which applied to what I had, and that every tool had like 10 alternatives.

Anyway that's it for this brain-dump. If you have questions about something I used
or why I didn't use something, send me an email.

Here is quite possibly the worst and most confusing diagram ever, yw:

![alt text](/assets/images/bad-network-pic "basic baremetal networking")

[flintlock]: https://github.com/weaveworks-liquidmetal/flintlock
[equinix]: https://metal.equinix.com/
[directlvm]: https://docs.docker.com/storage/storagedriver/device-mapper-driver/#configure-direct-lvm-mode-manually
[vlan]: https://www.youtube.com/watch?v=jC6MJTh9fRE&ab_channel=PowerCertAnimatedVideos
[bonded]: https://metal.equinix.com/developers/docs/layer2-networking/hybrid-bonded-mode/
[unbonded]: https://metal.equinix.com/developers/docs/layer2-networking/hybrid-unbonded-mode/
[l2]: https://metal.equinix.com/developers/docs/layer2-networking/layer2-mode/
[nat]: https://www.youtube.com/watch?v=FTUV0t6JaDA&t=1s
[tailscale]: https://tailscale.com/
[router]: https://tailscale.com/kb/1019/subnets/
