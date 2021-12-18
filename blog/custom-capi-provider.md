---
layout: post
permalink: /blog/custom-capi-provider
title: "Building and Shipping a custom Cluster API provider"
date: "December 17, 2021"
---

#### ... mostly shipping to be honest

## A guide for the uninitiated

At work I am currently getting a custom CAPI provider ready for production use.
I can't share it with you I'm afraid, since that repo is private at the time of
writing.

Luckily on my team we have someone who contributes heavily to [Cluster API
Provider AWS (CAPA)](https://github.com/kubernetes-sigs/cluster-api-provider-aws),
so they are able to answer my questions on how the hell I am supposed
to turn a thing that I can deploy locally into an official release that our
many fans can consume and enjoy.

Because the thing with CAPI is; its [docs](https://cluster-api.sigs.k8s.io/) are pretty good,
better than most really... but they are kindof all over the place. So once I got
to the end of the [Provider Implementers Guide](https://cluster-api.sigs.k8s.io/developer/providers/implementers-guide/building_running_and_testing.html),
I wasn't sure what I was meant to do to get my provider out into the wild.

And my google-fu wasn't helping. There are only a few official providers out there,
and maybe custom provider creating isn't that common? So nobody is really writing
about it, beyond the odd "getting started on your own machine" tutorial. Hence why,
when it came to shipping, I was left asking someone who had done it before and
looking at other providers' release processes (which are basically vast amounts
of yaml and Makefile... ew).

It wasn't until I searched for [`clusterctl` config](https://cluster-api.sigs.k8s.io/clusterctl/configuration.html)
(something I only did because my colleague said that was a thing we would
need to use) that I actually found pieces of the information I was looking for.

### CAPI?

This post assumes CAPI is not a new concept for you, but just in case, the TLDR is:
Cluster API is a framework which runs in Kubernetes. It lets you create more
clusters. You simply give it a configfile (like you would when deploying a regular
old k8s app), and 'boom' a cluster appears where you wanted it to, like on AWS,
for example. And you can create as many clusters as you like, using essentially
the same config barring a few name changes.

### Provider?

CAPI is 'pluggable', meaning the core pieces can be used to create clusters of
many different flavours, if you supply the extra parts. For example, to create
clusters on AWS, you would plug the AWS provider [CAPA](https://github.com/kubernetes-sigs/cluster-api-provider-aws)
into CAPI and that provider would go ahead and create all the AWS specific parts
of your cluster, like the EC2 instances and networking.
There are lots of types of Provider with CAPI, but I am talking about infrastructure
providers in this piece.

### Custom provider?

The theme of this post. Because CAPI is so pluggable, it is easy for you to write
your own components, plug them in, and have clusters created on whatever hot new
infrastructure is out there.

In my team's case we are creating an infrastructure provider which will create
k8s clusters on bare-metal.

![alt text](/assets/memes/baremetalhot.jpg)

What is not easy, or what is quite hard to discover, is how to get your custom
provider shipped.

### Creating a custom provider

I am not going to do a tutorial on this. I may come back for it later, but the
[CAPI docs](https://cluster-api.sigs.k8s.io/developer/providers/implementers-guide/overview.html)
for how to make one are good, so I recommend following that to create
your provider and coming back here after :D

### Releasing a custom provider

The above linked tutorial stops just after the build and deploy locally bit.
Now you want to ship. But what do you do? How do you give your users a nice
install experience? Making them clone your provider repo and run `kustomize` is
a little tacky.

CAPI management clusters are initialised with a tool called `clusterctl`. You'll
want to plug your provider into this tool to get this sweet user experience.

```bash
# bootstrap the management cluster
clusterctl init -i awesome-provider

# create a cluster using awesome-provider
clusterctl generate cluster -i awesome-provider:v0.1.0 cool-cluster-name | kubectl apply -f-
```

Wow.

Once I had been informed that `clusterctl` config was the thing to do, I found my way to the [relevant
docs](https://cluster-api.sigs.k8s.io/clusterctl/provider-contract.html).

So let's publish a release of that custom provider. We are going to do it manually
(I'll leave it to you to set up whatever slick automation for it) and we are
going to stick to the absolute bare minimum of requirements.

#### Push the image

First we are going to push that docker image that contains your provider `manager`
binary somewhere (this is the one you built in the tutorial linked above).
Dockerhub is fine. Tag it with something like `v0.1.0`.

```bash
# Your exact build command my vary, depending on what is going on with your provider
docker build . -t <your org>/<your image name>:v0.1.0
docker push <your org>/<your image name>:v0.1.0
```

#### Update the manifest

Next, update your `manager_image_patch.yaml` to have the correct image. That file
will be somewhere under the `config` directory. Mine is under `config/default`,
yours could be `config/manager`.

When you find your file, update the `image` field to match CAPI's canonical format:

```yaml
...
- image: docker.io/<your org>/<your image>:v0.1.0
...
```

#### Generate the component manifest

Next we are going to run the same `kustomize` command as in the CAPI tutorial,
but instead of applying anything we are just going to save a file.

```bash
kustomize build config/ > infrastructure-components.yaml
```

The `config/` path there may not work for you, depending on your provider setup.
If it doesn't, just give the path to the directory your `manager_image_patch.yaml`
is in, mine is `config/default`.

The filename `infrastructure-components.yaml` is really important: `clusterctl`
will be looking for it. There is no need to commit it unless you want to, we'll
get to using it later. If you are writing a different type of provider, this
file will have a [different name](https://cluster-api.sigs.k8s.io/clusterctl/provider-contract.html#naming-conventions).

#### Create a metadata file

This file will be used by `clusterctl` to know which version of CAPI your provider
is built to work with.

Create a file called `metadata.yaml` (most providers keep it in the root of their
project):

```yaml
# this is the version of clusterctl people will using to
# install with. usually set to whatever is latest
apiVersion: clusterctl.cluster.x-k8s.io/v1alpha3
# the major/minor bits are your provider's version record
# whenever you bump a maj/min you will add another section
# to this list. you don't need to add anything for patches
releaseSeries:
  - major: 0
    minor: 1
    # this contract is the version on capi that this maj/min
    # version of your provider is made to work with
    contract: v1beta1
```

The `v1alpha3` of `clusterctl` and the `v1beta1` of (the Cluster API) `contract`
here may vary depending on what you used for your provider and when you are
reading this. Update accordingly.

Commit and push this file to your provider repo.

#### Create a cluster template

This section is optional, as I am aware that the CAPI custom provider tutorial
does not include it. Seems this is another "just need to know it" thing.
I will come back and add how to do that later, or maybe contribute that piece
to the CAPI docs.

If you do want to follow their more advanced docs on [ClusterTemplates](https://cluster-api.sigs.k8s.io/developer/providers/cluster-infrastructure.html#infraclustertemplate-resources)
and [MachineTemplates](https://cluster-api.sigs.k8s.io/developer/providers/machine-infrastructure.html#inframachinetemplate-resources),
please go ahead, and store the template in `templates/cluster-template.yaml`.

Templates are what let your users do this:
```bash
clusterctl generate cluster -i awesome-provider:v0.1.0 cool-cluster-name | kubectl apply -f-
```

Without that they will have to create their own yamls (or you will have to give
them some nice yamls to download) to create a cluster built by your provider.

#### Create the release

In Github (or Gitlab, this guide is assuming Github, but of course release
wherever you like) , navigate to the repo containing your custom provider.

Click `Create new release`. From the `Choose a tag` dropdown, set `v0.1.0`
and click `+ Create new tag`.

Put `v0.1.0` as the release title (or whatever you want really, but naming
releases as basic semver tags is standard across CAPI).

Attach the files we just created:
- `infrastructure-components.yaml`
- `metadata.yaml`
- `templates/cluster-template.yaml` (if you have it, no worries if not)

And that's it; create your release

#### Deploy your release

Now you, and your many adoring fans, can easily deploy your provider into their
management cluster and create lots of little child clusters with it.

The last thing we need to do is wire it up with your local `clusterctl`.

Create a file `~/.cluster-api/clusterctl.yaml` and write the following to it:

```yaml
providers:
  - name: "<your provider name>"
    url: "https://github.com/<org>/<repo name>/releases/latest/infrastructure-components.yaml"
    type: "InfrastructureProvider" # change this if you are building another provider type
```

And now all we do is:

```bash
clusterctl init -i <the provider name you put in the file above>
```

Those last 2 steps, the file and the command, is what goes in your user docs.

If you also uploaded a template file, you can instruct your users to create a
cluster like so:

```bash
clusterctl generate cluster -i <provider name>:v0.1.0 <cluster name> | kubectl apply -f-
```

But if not, you can simply supply them with some example yaml.

&nbsp;

------------------

&nbsp;

And that's it! You have now published a custom CAPI provider; I hope your users
appreciate you for it.

I mainly wrote this for my own use, so that I didn't forget, but hopefully
it is helpful for someone else too. Get in touch if you have any questions, thanks
for reading :wave: .
