---
title: Vagrant on Windows with gulp-watch and similar file watching tools
date: 2016-02-16 00:00:00 Z
tags:
- Vagrant
- gulp-watch
---

As you might have guessed from my previous posts I love Vagrant and Gulp. I think those tools are
awesome. They make my life much easier by either packing everything into a VM which I can destroy
and recreate whenever things go south (Vagrant), or automating every little bit I need (Gulp).
However, until recently, I didn't use them together. Vagrant was there to get my servers (mostly DB)
up and running and Gulp was directly on my machine.

**TL;DR;** - skip down to **Vagrant RSync Problems and Solutions** and further **Solutions**.

# Why file watchers and Vagrant don't like each other?

Well, it's not Vagrant directly, it's VMs in general. `gulp-watch`, `jekyll serve` or `meteor`
need to watch your file system in order to detect changes and start acting upon them. This works
usually great. Usually meaning, until you want to run those tools inside a VM and access files
stored directly on your machine. Vagrant does a lot of magic to keep your local folder in sync
with the VM, but ultimately it chooses one of these types to sync the folders:

## [VirtualBox](https://www.vagrantup.com/docs/synced-folders/virtualbox.html)

This is default when you are running Vagrant with a VirtualBox and it works great. They made this
work when I couldn't really set it up in VirtualBox directly and something was breaking all the time.
This is awesome and you probably rarely, if ever have problems with it. That is, until you try
using `gulp-watch` or similar from the VM to watch changes you are performing locally, as
file watchers don't work in this configuration. So let's see what else Vagrant is offering.

## [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html)

The limitation for this is written right in the second paragraph:

> **Windows users:** NFS folders do not work on Windows hosts. Vagrant will ignore your request for NFS synced folders on Windows.

Although file watchers should be working with NFS, it kinda defeats the purpose. Since the reason I
desperately want to get tools which need file watchers running inside Vagrant is,
that those tools are probably written for Unix systems (those systems where NFS works),
but not for Windows. Maybe you remember, there was a time when [Meteor](https://www.meteor.com/)
didn't work on Windows. This would have been the perfect way to get it to work, if NFS would be working.

## [SMB](https://www.vagrantup.com/docs/synced-folders/smb.html)

This is a new one. I don't remember this being around, when I tried to get `gulp-watch` working inside
the VM. It seems to be like NFS for Windows:

> **Windows only!** SMB is currently only supported when the host machine is Windows. The guest machine can be Windows or Linux.

This sounds like great news, but it gets more discouraging when you keep reading. At first they list
all the requirements necessary on host and guest machines and in the end finish it up with this lines:

> Because SMB is a relatively new synced folder type in Vagrant, it still has some rough edges. Hopefully, future versions of Vagrant will address these.

Still, this is something you probably want to give a try from time to time, until you find out it is working.

## [RSync](https://www.vagrantup.com/docs/synced-folders/rsync.html)

The last option, RSync. This isn't magic this is relatively dumb. You need to do everything by hand.
First of all, you need to have `rsync.exe` available on your local machine. You can install it either
directly or you can get it with some [Cygwin](https://www.cygwin.com/) environment. Then you need
to open an extra console / shell / command line to run the following command:

    vagrant rsync-auto

And this command needs to keep running all the time. It will be watching your local file system and
use the rsync protocol to transfer those changes over the virtual network into the VM, where the
files will be stored locally. If you think about it at first, it doesn't sound so bad. The files
will be locally accessible in the machine and therefore all file watchers only need to watch the VM
file system, not the host machine anymore. It sounds great, soon you'll find out all the problems you'll get...

# Vagrant RSync Problems and Solutions

We already started with the problems. It's an extra command which needs to keep running all the time.
It's not a big deal, but it just doesn't seem right. Unfortunately, this is not everything:

## Problems

- RSync works only one way. There is no way to get the files out of the VM again with this configuration.
  Of course, you can still get them out, the changes done inside the VM just won't end up magically on your local machine.
- RSync tries to have an exact clone of your host folder on the VM. This sounds good at first,
  until you realize what that actually means:

  - If you are running `gulp-watch` you need folders like `node_modules` RSync will be trying to keep them up to date with the host folder.
  - Just excluding folders doesn't work either, because that means for RSync that those folders shouldn't exist in the destination (guest folder).

- And in the end, it takes quite a while to see the changes, because you have 2 additional steps in between. Just think about it:

  1. You save a change to a file
  2. `rsync.exe` picks up this change on your local machine and updates the destination
  3. These changes are written to the file system on the VM
  4. Now only `gulp-watch` or the tool you are running can pick up these changes and start processing them.

## Solutions

I don't have a solution to everything and for someone with a lot of experience with RSync, those problems
mentioned above wouldn't really qualify as such. Unfortunately, I didn't have too much experience
with RSync up to now, so I had to learn it the hard way. Here some examples from my Vagrantfile:

    config.vm.synced_folder ".", "/vagrant", type: "rsync",
      rsync__exclude: [".git/", ".vagrant/", "node_modules/", "bower_components/", ".tmp/", "dist/"],
      rsync__args: ["--verbose", "--archive", "--delete", "-z", "--copy-links",
                    "--filter=P node_modules/**/*",
                    "--filter=P bower_components/**/*",
                    "--filter=P .tmp/**/*",
                    "--filter=P dist/**/*"]

As you can see, I'm excluding some folders from being copied, and also being watched by RSync. The
later is the reason `.git` is excluded. Afterwards, I'm protecting (`--filter=P`) those folders,
which I don't want to get deleted inside my VM.

That's really it. This is how you can make RSync work for you and enjoy some of the packages which don't
yet quite work on Windows natively inside a VM without sacrificing watchers. It was just very painful
to get to this state of knowledge. Hope I could save you some pain.
