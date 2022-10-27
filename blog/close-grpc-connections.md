---
layout: post
permalink: /blog/close-grpc-connections
title: "Always close gRPC client connections"
date: "August 18, 2022"
---

TLDR: close your goddamn client connections or face the wrath of system open file limits!

&nbsp;

---------------------------------------------

&nbsp;

When you follow a tutorial on "how to build a grpc client", it will often look
something like this:

```go
conn, err := grpc.Dial("localhost:8080", grpc.WithTransportCredentials(insecure.NewCredentials()))
if err != nil {
  // some error handling
}
defer conn.Close()

c := client.NewClient(eg.NewBasicClient(conn), os.Stdout)
c.SomeMethod()
```

There may be a line saying "Make sure you remember to close the connection" (that's
the `defer conn.Close()` bit, the `defer` makes sure it is the last thing that
always happens when the function exits). Or maybe there wont be, and it is taken
as given that whoever is reading will either "just know" to do it, or will be
copy-pasta -ing the whole thing, which in this case is not so terrible.

But those of us who have always "just" thrown in the `defer` as the "right thing"
to do may not know fully why it is the right thing. What exactly happens when
clients, long-running clients in particular, don't close their connections to the server?

Normally I find out what a thing does be removing it and observing the chaos.
In this case, the thing was already removed, and it presented in a interesting bug
which took us a moment to trace back to the un-disconnected clients.

&nbsp;

---------------------------------------------

&nbsp;

I had been messing around, building some spikes for possible usecases of [Liquid Metal][lm],
when I noticed that often I would come back to my environment the next day to
find that my [flintlock][flint] (gRPC) server had frozen up.

Client requests from my [CAPMVM][capmvm] controller and from my little CLI tool,
[hammertime][ht], would hang for a bit, then eventually error with:

```
rpc error: code = Unavailable desc = failed to receive server preface within timeout
```

which told me nothing, and Google didn't give me anything either. The most you
can glean from that error is that the service is still up, but is so jammed that
it cannot respond with the most basic `ack` to any client.

At the time, I had other things to do, so I noted the [issue][issue] and restarted my
flintlock server, which made things work again for a time.

We then got a report from someone in CX that they were seeing the problem on their
long-running Liquid Metal environments too, so we both took a morning to investigate.

It turned out that my environment was back in that state again, so I started to
poke around, while the other member of my 2-person team, the venerable [Richard Case][rc],
focussed on getting a faster reproduction going.

After checking the basics (service status, flintlockd logs, process/thread stacks, etc),
I took a look at the network:

```bash
$ netstat -a | awk '/:9090/ && /ESTABLISHED/'
tcp6       0      0 host-0:9090             host86-183-165-13:37492 ESTABLISHED
tcp6       0      0 host-0:9090             host86-183-165-13:60536 ESTABLISHED
tcp6       0      0 host-0:9090             host86-183-165-13:39056 ESTABLISHED
tcp6       0      0 host-0:9090             host86-183-165-13:41894 ESTABLISHED
...
```

and so on for 1012 lines.

Basically there were over a thousand open connections to my `flintlockd` server,
which had hit the open file limit set by the system I was running on.
I guessed that restarting the service had made the problem go away because all
of those connections would have been closed.

All the connections went to the same IP which was likely my CAPMVM controller,
which would have been opening (and not closing) several connections per reconciliation
loop.

This time I closed them all without restarting, in case the restart was masking
something else.

```bash
# netstat -a = get all connections
# awk '/:9090/ && /ESTABLISHED/ {print $5}' = filter for connections established
#   on port 9090 (flintlock) return the fifth column, which is where the connection
#   is coming from
# cut -f 2 -d ":" = split the fifth column at the ':' get the 2nd part which is the
#   port of the connection
# xargs -I {} = use the output of the previous command as the arg to the next command
#   mark the arg with '{}'
# ss -K dport {} = kill the connection to this port

$ netstat -a | awk '/:9090/ && /ESTABLISHED/ {print $5}' | cut -f 2 -d ":" | xargs -I {} ss -K dport {}
Netid                 State                 Recv-Q                 Send-Q                                           Local Address:Port                                            Peer Address:Port                 Process
tcp                   ESTAB                 0                      0                                      [::ffff:147.75.102.155]:9090                                 [::ffff:86.183.165.138]:46442
... # lots of output
```

After this I re-ran my `hammertime` command and it went through fine. The un-closed
connections were the culprit.

_Meanwhile..._

Richard had come to the same conclusion via a different symptom at about the same time. For his part,
he had written a small client which would loop, open a connection to flintlock,
NOT close it, and call an endpoint until it couldn't anymore. On the other side
he had started a custom flintlock server in which he had enabled [`pprof`][pprof].

While the client tool ran, he saw the number of server `goroutine`s (and `allocs`)
increase until the server stopped processing requests and the client started
receiving the `"failed to receive server preface"` error.

![alt-text](/assets/images/pprof-before.png "pprof with connections left open")

When he rebuilt the client to close the connections, the goroutines stayed stable,
and the server continued to serve requests without issue.

![alt-text](/assets/images/pprof-after.png "pprof with connections closed")

Note that here the maxing out of allowed open connections and the number of goroutines
are likely to be unconnected. Either flintlock had so many routines open that it
couldn't process more, or the system had said "no more open files" which meant flintlock's
hands were tied.

Either way, leaking that many routines or opening that many connections is just careless.

It is also interesting to note that this bug only appeared because of the way we
were communicating with the service:
- It was a long-running sub-routine
- Which did not exit
- And which opened a new connection (or three) per reconciliation loop

If the client had re-used the same connection throughout, or if it exited after calls
thus disconnecting, we would not have seen the issue.

&nbsp;

---------------------------------------------

&nbsp;

Most often the "right thing" has a good reason for being "right", and it was nice
to get to see the reasoning and repercussions of this one.

Basically: whatever you are opening (files, connections, the fridge), close it.
With a `defer` if you have one.

For a full repro of this bug, as well as an illustration of how to set up `pprof`
on your service, I have set up a dummy service and client [here][soda].

[lm]: https://github.com/weaveworks-liquidmetal
[flint]: https://github.com/weaveworks-liquidmetal/flintlock
[capmvm]: https://github.com/weaveworks-liquidmetal/cluster-api-provider-microvm
[ht]: https://github.com/warehouse-13/hammertime
[issue]: https://github.com/weaveworks-liquidmetal/flintlock/issues/503
[rc]: https://github.com/richardcase
[pprof]: https://github.com/google/pprof
[soda]: https://github.com/warehouse-13/soda
