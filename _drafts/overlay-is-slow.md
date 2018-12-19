---
layout: post
title: "Overlay is Slow(er than before)"
---

## **TL;DR:**

_Spoiler alert!!_

Various changes in how the kernel triggers writeback of dirty inodes introduced between v4.4 and v4.15
led to a marked decrease in performance when it came to handling our Overlay mounted container root filesystems.

Fortunately, because of luck or magic, we may have already solved* the problem long before we knew we had it.

_\*got around_


## **The Full Story**

So, one day towards the end of September, we looked up at our team CI/Performance monitor and saw this:

![alt text](/assets/images/cfscalegraph "one of our monitoring graphs")

I bet that, despite having been given limited context and perhaps not even knowing what my team does,
you looked at that graph and thought "oh damn". The reason you thought this (maybe not exactly this) is because everyone knows that
sudden changes in graphs either mean something very good just happened or something very not good just happened.

This was the second thing.

I'll go back a bit. My team provides the container runtime for the open source PaaS, [Cloud Foundry](https://www.cloudfoundry.org/). Essentially what
this boils down to is: when application developers push their code to the cloud, after various things going through
various components, we will ensure that app code is booted inside a secure and isolated environment. And we will ensure that it
is booted _quickly_. Speed is, after all, one of the many reasons we love and use containers for this sort of thing.

That is what this graph is showing. Ignore the blue line for now, I will come back to it later because it _is_ important but
until then just focus on the purple one, the one showing how suddenly Not Fast we are at booting peoples' apps for them.

Naturally, the first thing we did was find out what had changed in our various repos. A quick `git blame` revealed... my name.

But I hadn't actually done anything to our code. What that commit did was change the default environment our components run on.

In the last year, all Cloud Foundry components have been moving from Ubuntu Trusty 14.04 based VMs, to Ubuntu Xenial 16.04 ones. Our team
has been testing against both for a while to ensure compatibility, with a heavy preference for Xenial since that would soon be the only option.
I just happened to notice on that fateful day that the VMs we were using to get the data for that particular graph were provisioned with Trusty, so I
thought it sensible to switch them over.

This turned out to be a good idea: it would have been very embarrassing to have got all the way to prod (at this time not yet on Xenial) without
noticing an up to 10s increase in application start times.

So what is this graph actually showing? It is not showing an individual application create nor is it showing just the container components.
It is measuring the application developer's whole experience of scaling an already running application from 1 to 10 instances.

The abridged sequence of events is as follows:
- The asynchronous request from the app developer (via CLI) goes to the Cloud Controller
- The Cloud Controller forwards the request to the Diego-Cell (the scheduler)
- The Rep component in the Cell asks Garden-Runc (that's us!) to create 9 new containers
- Garden-Runc creates those containers
- The Executor component streams the application code into the containers
- The Rep asks Garden-Runc to create a sidecar process for each container
- Garden-Runc creates those sidecar processes, which check whether the app has booted successfully
- When all of those processes exit successfully, Rep reports that the app instances are ready to the Cloud Controller
- The Cloud Controller reports back to the CLI
- The application developer sees they have 10 running instances and is happy

It is the time taken for all of this which is plotted in that graph.

With so much going on over which we have no control, it was very possible that the slowdown was not down to us at all. But knowing our luck
and our track record with major environmental shifts, honestly it was always going to be us.

# **1: Logs**

Obviously we started with our logs. Garden-Runc components log _a lot_, so we started by pushing an app to our environment, grepping all mentions of that
handle in `garden.stdout.log`, and having a good ol' read. And what we saw was:

```
{"timestamp":"2018-10-24T11:39:55.231873213Z","message":"guardian.run.create-pea.clean-pea.image-plugin-destroy.grootfs.delete.groot-deleting.deleting-image.overlayxfs-destroying-image.starting","data":{"handle":"2d8f7e6d-66fa-4e5d-5cc1-da07-readiness-healthcheck-0"}}
{"timestamp":"2018-10-24T11:40:03.189979808Z","message":"guardian.run.create-pea.clean-pea.image-plugin-destroy.grootfs.delete.groot-deleting.deleting-image.overlayxfs-destroying-image.ending","data":{"handle":"2d8f7e6d-66fa-4e5d-5cc1-da07-readiness-healthcheck-0"}}
```
_Thanks to Nima Kaviani for PRing human-readable timestamps and saving me a lot of tediousness!_

Why are we deleting an image? What is an image? What is a grootfs? What is a "pea"? Why does it need cleaning? Aren't we meant to be creating things here?

There is a lot of info in these logs (even though I have edited them for you), so let's break them down.

Guardian and GrootFS are components of [Garden-Runc](https://github.com/cloudfoundry/garden-runc-release). [Guardian](https://github.com/cloudfoundry/guardian) is our container creator. It prepares an OCI compliant container spec, and then passes that to [runc](https://github.com/opencontainers/runc),
our container runtime, which creates and runs a container based on that spec. Before Guardian calls runc, it asks [GrootFS](https://github.com/cloudfoundry/grootfs) (an image plugin) to create a root filesystem for the
container. GrootFS imports the filesystem layers and uses OverlayFS to combine them and provide read-write mountpoint which acts as the container's rootfs. The
path to this rootfs is included in the spec which Guardian passes to runc so that runc can [pivot_root](http://man7.org/linux/man-pages/man2/pivot_root.2.html) to the right place.

(_for more on container root filesystems, see [Container Root Filesystems in Production](/2017/09/08/container-rootfilesystems-in-prod)_)

A Pea is our name for a sidecar container. (Docker groups their sidecars into a Pod, as in a pod of whales. We are Garden so our sidecars are peas... in a pod.
Yes, it is hilarious.) Sidecar containers are associated with "full" containers and share some of the resources and configuration of their parent, but can be limited in a different way.
Crucially, in the context of our problem, they have their own root filesystem. In the case of the logs above, the sidecar (pea) was created to run a process which checks
whether the primary application container has booted successfully. When the process detects that the app is up and running, it exits, which triggers the teardown
of its resources, including the rootfs.

So that is what we are seeing here: for whatever reason we have yet to discover, after a pea has determined that our developer's app is running and goes into teardown,
GrootFS is taking ~8s to delete its root filesystem.

Given that this slowness appeared right after I bumped our VMs' OS, which we know included a jump from a 4.4 kernel to a 4.15 kernel, I was immediately Very Suspicious
that we were apparently losing time in a rootfs deletion.

But there is more to a GrootFS rootfs (_sigh, naming_) delete than a unmount syscall, so we needed a little more evidence before wading into a kernel bisection.

# **2: More Logs**

So we added even more log lines to GrootFS and pushed another app.

```
{"timestamp":"2018-10-25T13:12:25.314306368Z","message":"...deleting-image.overlayxfs-destroying-image.starting","data":{"handle":"6a0aed04-fe8e-47a2-5d0b-dfbfc165534f-readiness-healthcheck-0"}}
{"timestamp":"2018-10-25T13:12:25.331496182Z","message":"...deleting-image.overlayxfs-destroying-image.getting-project-quota-id","data":{"handle":"6a0aed04-fe8e-47a2-5d0b-dfbfc165534f-readiness-healthcheck-0"}}
{"timestamp":"2018-10-25T13:12:25.333481923Z","message":"...deleting-image.overlayxfs-destroying-image.project-id-acquired","data":{"handle":"6a0aed04-fe8e-47a2-5d0b-dfbfc165534f-readiness-healthcheck-0"}}
{"timestamp":"2018-10-25T13:12:25.335395181Z","message":"...deleting-image.overlayxfs-destroying-image.unmounting-image","data":{"handle":"6a0aed04-fe8e-47a2-5d0b-dfbfc165534f-readiness-healthcheck-0"}}
{"timestamp":"2018-10-25T13:12:32.528611226Z","message":"...deleting-image.overlayxfs-destroying-image.image-unmounted","data":{"handle":"6a0aed04-fe8e-47a2-5d0b-dfbfc165534f-readiness-healthcheck-0"}}
{"timestamp":"2018-10-25T13:12:32.529112053Z","message":"...deleting-image.overlayxfs-destroying-image.removing-image-dir","data":{"handle":"6a0aed04-fe8e-47a2-5d0b-dfbfc165534f-readiness-healthcheck-0"}}
{"timestamp":"2018-10-25T13:12:32.549213494Z","message":"...deleting-image.overlayxfs-destroying-image.image-dir-deleted","data":{"handle":"6a0aed04-fe8e-47a2-5d0b-dfbfc165534f-readiness-healthcheck-0"}}
{"timestamp":"2018-10-25T13:12:32.550306115Z","message":"...deleting-image.overlayxfs-destroying-image.ending","data":{"handle":"6a0aed04-fe8e-47a2-5d0b-dfbfc165534f-readiness-healthcheck-0"}}
```

[The code](https://github.com/cloudfoundry/grootfs/blob/afb5cee1eb3b8c767f6a07c0b36a9f36e666d2dd/store/filesystems/overlayxfs/driver.go#L683-L685) between those two log lines was doing a `syscall.Unmount` and nothing more. 

Now that this was looking like a kernel thing and not a Garden/CF thing, we needed to tighten our testing loop.

# **3: Reliable Reproduction**

At the Garden level, and certainly at the GrootFS level, there is no knowledge of an "application". To Garden a scaled application is the same as an initial application: all
it sees is containers. Peas (sidecars) are handled a little differently by Guardian and therefore follow their own code divergence, but to Runc (the thing handling the creation)
a Sidecar container is just another container. Garden may think it has created 10 containers with 10 sidecars, but Runc knows it has simply created 20 containers.

Going further, what does GrootFS know? GrootFS has the least awareness of anything containery. It just simply, on request, creates a mounted directory which could maybe be used as a root filesystem.

Given that a syscall in heart of GrootFS has the most incidental and tenuous connection to a CF application, we were able cut out a great many extraneous parts and just use
a small bash script to mimic an application scale at the filesystem level:

- serially create 10 overlay mounts
- unmount all mounts simultaneously

... and time each unmount to see some unreasonable slowness.

But it showed nothing. Unmounts went through in under a second. Were we wrong?

Looking back through the logs, we remembered that, of course, a container rootfs wouldn't just be mounted and then unmounted: code would be
streamed in. We amended our script to include writing a file (approximately the size of the app we used earlier) into the mountpoint.

And this time we got what we were after: each unmount took up to 10 seconds to complete.

The problem presenting only when data had been written to the mount backed up our theory that this was kernel related, specifically the overlay filesystem, which
has been built into the mainline since version 3.18. Given that an [overlay filesystem](https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt) can have up to 3 different [inodes](http://www.grymoire.com/Unix/Inodes.html) for any given file (one at the mount, one in the lower layer
and one in the upper layer if the file has been written), we hypothesised that perhaps something in the way inodes were allocated in overlay had changed between 4.4 and 4.15.

To prove this we would have to use [our script](/assets/gists/overlay-umount-test) to determine which kernel first saw this problem.

# **4: Narrowing It Down...**

We were not quite at the point where we needed to do a proper kernel bisect, fortunately there are a lot of pre-compiled versions [available online](http://kernel.ubuntu.com/~kernel-ppa/mainline/),
so we were able to jump between those in what I like to call a _lazy bisect_.

As we were fine on 4.4 and not fine on 4.15, we picked a version sort-of in the middle to start: 4.10.

Switching out a kernel is a [fairly straightforward thing](/assets/gists/switch-kernel). Canonical puts all the [debs online](http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10/), so once we downloaded those,
we could simply install, update grub, reboot and wait. After a little time we were able to ssh back on and run our reproduction script.

Unmounts went through in a flash, so we moved on to 4.13 (sort-of in between 4.10 and 4.15), ditto, then bumped up to 4.14.
This time we saw what we were after: slow overlay unmounts.

So 4.14 was the first minor version on which slow overlay unmounts could be observed. Now we kept going to discover which patch brought it in.

The patch versions in 4.14 run from 1 to 78, so we again started somewhere vaguely in the middle: 4.14.39. Nothing there. 4.14.58? Nope.
4.14.68 and we were back in business: slow unmounts. 4.14.63 was also slow, and so was 4.14.60.

Our last pre-compiled run was 4.14.59, which showed no performance regression, so now we knew that we were looking for something which went into 4.14.60.

# **5: Mainline Crack**

We were now out of pre-compiled kernel debs so we needed to roll our own.

The first step was to clone down the [ubuntu mainline kernel repository](https://git.launchpad.net/~ubuntu-kernel-test/ubuntu/+source/linux/+git/mainline-crack)
(if you plan on trying this at home, be warned: cloning takes ages because the repo is mega). When it finally downloaded,
we looked at the git log to see which commits went into our problem version.
Normally when bisecting something, you would start with a commit in the middle and then work your way up or down until you get to the culprit.

But since simply switching between pre-compiled versions was slow, I hoped we could cheat a bit to get around compiling that much. So we read
the [messages of the commits](http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.14.60/CHANGES) which went into 4.14.60 to look for likely candidates which did things to anything filesystemy, fs/overlay especially.

[One message](https://lore.kernel.org/patchwork/patch/969996/) jumped out at us immediately: `ovl: Sync upper dirty data when syncing overlayfs`.
The rest of the message also looked good:

>When executing filesystem sync or umount on overlayfs,
>dirty data does not get synced as expected on upper filesystem.
>This patch fixes sync filesystem method to keep data consistency
>for overlayfs.

So we decided to focus on this commit first, and go to a regular old bisect from there if it didn't work out.

We [created two custom kernels](/assets/gists/kernel-bisect), one at that commit, and one at the commit before. We tested against the suspect commit first, because that was just the sensible thing
to do. The unmounts were slow, and it felt like we might get away with it, but we did our best not to get excited. We switched to the commit before, rebooted, and ran our tests.

All ten overlay unmounts go through in under a second.

Woohoo! Our gamble paid off, which is a relief because while it took you perhaps a minute to read the last two sections, that whole nonsense took us a week.

# **6: sync_filesystem**

So what happened in that commit?
Let's take a look at the diff:
```diff
--- a/fs/overlayfs/super.c
+++ b/fs/overlayfs/super.c
@@ -232,6 +232,7 @@ static void ovl_put_super(struct super_b
 	kfree(ufs);
 }
 
+/* Sync real dirty inodes in upper filesystem (if it exists) */
 static int ovl_sync_fs(struct super_block *sb, int wait)
 {
 	struct ovl_fs *ufs = sb->s_fs_info;
@@ -240,14 +241,24 @@ static int ovl_sync_fs(struct super_bloc
 
 	if (!ufs->upper_mnt)
 		return 0;
-	upper_sb = ufs->upper_mnt->mnt_sb;
-	if (!upper_sb->s_op->sync_fs)
+
+	/*
+	 * If this is a sync(2) call or an emergency sync, all the super blocks
+	 * will be iterated, including upper_sb, so no need to do anything.
+	 *
+	 * If this is a syncfs(2) call, then we do need to call
+	 * sync_filesystem() on upper_sb, but enough if we do it when being
+	 * called with wait == 1.
+	 */
+	if (!wait)
 		return 0;
 
-	/* real inodes have already been synced by sync_filesystem(ovl_sb) */
+	upper_sb = ufs->upper_mnt->mnt_sb;
+
 	down_read(&upper_sb->s_umount);
-	ret = upper_sb->s_op->sync_fs(upper_sb, wait);
+	ret = sync_filesystem(upper_sb);
 	up_read(&upper_sb->s_umount);
+
 	return ret;
 }
```

The key thing we care about here is this new line which is treating that upper superblock differently: `ret = sync_filesystem(upper_sb);`.
What is `sync_filesystem` doing? A fair amount, and I'm not gonna paste the code in, you can [check it out here](https://github.com/torvalds/linux/blob/v4.15/fs/sync.c#L24-L69),
but we are going to look at some choice comments:

```
/*
 * Write out and wait upon all dirty data associated with this
 * superblock.  Filesystem data as well as the underlying block
 * device.  Takes the superblock lock.
 */
```

A superblock lock? We may not have known precisely what was going on yet, but we now could perhaps see why _all_ unmounts took such a long time: whichever call gets there
first is claiming this lock while the others wait their turn.

We were able to verify this by editing our reproduction script to perform our unmounts in serial rather than in parallel. Sure enough, we saw the first unmount hang
for ~8 seconds and when it was done the others, with no dirty inodes (meaning new data which has not been written to disk yet) to sync and therefore no need to hang onto that lock, completed in <0.03 seconds.

But what is the first call doing which leads it to hold a lock for, in some extreme cases, nearly ~10 seconds? We can see that `sync_filesystem` in turn calls `__sync_filesystem` which, depending
on the value of `wait` will do _something_ with those dirty inodes. It may even take both courses of action should the first call not return `< 0`.

We could also see that the more mountpoints there were, or the more data was written to those mounts, the longer the sync took to flush the data: changing either
the number of mounts created to 20 or the amount of data written in to 20 counts would increase the locked time to ~20 seconds.

At this point, the Garden team pulled back: we had determined the point in the kernel which was slowing down our umounts as well as the commit which had introduced the key change.
We could tell that our key commit was doing The Right Thing and that the call to `sync_filesystem` was necessary, but much as we would have loved to
dig deeper ourselves, we are not kernel developers. Therefore we felt this was the correct point for us to return to the problems we _could_ fix, and write this particular one up for those
who had the skills to find the correct solution.

Fortunately, Pivotal has a close relationship with [Canonical](https://www.canonical.com/), so we were able to open a support ticket and get some expert help.

Our main questions were as follows:
- Could the modified `ovl_sync_fs` function be modified further to increase efficiency?
- Were we hitting some sort of threshold which determines when dirty inode data is synced to disk?

One last question, which had been bothering me for a while, answered itself as I began to write this blog: Why umounts?
Why did we not see this during any other Overlay operation, when there is nothing specific to unmounting about the `ovl_sync_fs` function?
It turned out this could be seen during other Overlay operations. While writing this, I created a new environment so that I could run my script and
get some good output. To my complete and utter panic, I saw that the same script I had been reliably running through multiple kernel switches for two weeks
was no longer producing the expected results. The unmounts were taking less than 0.1 of a second! I was immediately furious that I had wasted so much time being
completely wrong and honestly thought I would have to start over. But when I stopped flipping my desk over and actually watched the output of my script as it came through,
I noticed that around the 7th mount, the output paused for a good long moment.

When I adjusted my script to create just 6 mounts, I saw what I had expected to see: slow unmounts. So the inode sync could occur during other Overlay operations, but
what was different about this new environment which had altered the behaviour? I was now even more curious to hear what Canonical had to say about my second question above.

# **7: The Real Culprit**

_From now on the vast majority of the work and any cool discoveries made are down to Ioanna-Maria Alifieraki of Canonical._

Please check out our open source tracker to see all the nitty gritty of [our investigation](https://www.pivotaltracker.com/n/projects/1158420/stories/160845774) and the one [continued by Canonical](https://www.pivotaltracker.com/n/projects/1158420/stories/162409874).

# **8: The Blue Line**