---
title: "Vagrant + Docker = Rainbows + Unicorns"
tags:
  - Vagrant
  - Docker
---

# Step 0 - rant

Be honest with yourself, how many applications and services do you have running on your machine right now?
And how many do you really need?

2 weeks ago, when I started my new job, I remembered the times when I just had to fill up my
development machine with dozens of seemingly useless services. And of course I wouldn't remember all
of them, so I better made sure they all start up at automatically. Otherwise, I'd waste an hour after
every restart trying to figure out which services should be running. Also there isn't really a point
in starting them up manually, as I'd never shut them down again anyway.

The services I'm talking about are SQL Server, IIS and similar. They all run only on Windows and I
just don't have enough experience in order to do anything about it. But when I was told, I also should
install Elasticsearch on my machine, then it was enough. I decided, I'm not going to install a tool
which runs perfectly fine in Linux on a Windows machine and let it eat my resources all the time.

So what is the easiest way to get any common service or software up and running in no time? - **Docker**!
You just go to the [Docker hub](https://hub.docker.com/) and you look for what you need. In my case
an [Elasticsearch image](https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=elasticsearch&starCount=0).
You see the first image in the search results has the description *official*. That sounds great, I'll take that.

> Note, not every software will have an official image. That doesn't mean you shouldn't use it.
In most cases, the Dockerfiles (the instructions how the image is created) are open, so have a look at them.
If they seem to be valid, use them. If not, copy / fork them and adjust. Afterwards push it back to
Docker hub and you get your own custom image.

Back to our Elasticsearch image: [https://hub.docker.com/_/elasticsearch/](https://hub.docker.com/_/elasticsearch/)

So, how do we get this up and running?

# Step 1 - get VM with Docker

The easiest way I know to get a VM is [Vagrant](https://www.vagrantup.com/). At first, the tool was
very confusing for me. Why would I need a tool which controls VirtualBox? But as I learned to use
and love the Shell and had to provide VMs for colleagues for development, I started understanding the need.

So start up a Shell / Terminal / Command Window / however you call it and type:

    vagrant init box-cutter/ubuntu1404-docker

This gives us a very large Vagrantfile with a lot of comments.

# Step 2 - get the Docker image

Ok, open the Vagrantfile in your favorite editor and let's modify some of the last lines:

    # config.vm.provision "shell", inline: <<-SHELL
    #   sudo apt-get update
    #   sudo apt-get install -y apache2
    # SHELL

First, we'll need to uncomment them, and we don't want to run `apt-get update` or similar. Instead
we want to launch the Docker container here. So we get something like this:

    config.vm.provision "shell", inline: <<-SHELL
      docker run --name="elasticsearch" \
                 --restart="always" \
                 -p 9200:9200 \
                 -d elasticsearch
    SHELL

You see, `--restart="always"` makes sure that our Docker container will be started automatically,
every time we boot up the VM.

# Step 3 - forwarding the ports

Actually we are done here and you could run `vagrant up` to have Elasticsearch, or your favorite
service up and running in no time. But since we want to access it from the local machine, as if it
were installed locally, we'll need to forward the ports.

Search in the Vagrantfile for `forwarded_port` and you should find something like this:

    # config.vm.network "forwarded_port", guest: 80, host: 8080

This is a good template. Since Elasticsearch is using port 9200 and our container also exposes
it as 9200, we just need to make it the following line:

    config.vm.network "forwarded_port", guest: 9200, host: 9200

Now you can run `vagrant up` and it's basically done.

# Step 4 - brag how this is better

As a small add on - this has 3 major advantages over installing it directly on my Windows machine:

1. It is faster than setting up Java and installing Elasticsearch. Especially, because I don't need to press any additional buttons - `vagrant up` can run unattended.
2. I can start and stop it whenever I need. If I needed multiple services, I could combine them in the same VM and they'd all start and stop with one command.
3. If I need to update some service, I just update the container, no big deal. And if that doesn't work, I just run `vagrant destroy -f` and `vagrant up` again.

# Step 5 - icing on the VM

Let's go back to the Vagrantfile and see what we can modify. For instance,
I don't feel comfortable with the VM eating 512MB memory by default.
I have a VM with Postgres in a container only taking 256MB, let's adapt that. Find the following configuration part:

    # config.vm.provider "virtualbox" do |vb|
    #   # Display the VirtualBox GUI when booting the machine
    #   vb.gui = true
    #
    #   # Customize the amount of memory on the VM:
    #   vb.memory = "1024"
    # end

and change it to look like this:

    config.vm.provider "virtualbox" do |vb|
      # Customize the amount of memory on the VM:
      vb.memory = "256"
    end

This is great. But what about my MacBook? I don't want to use VirtualBox.
I'm already paying for Parallels and it has way more features. That's not a problem. Our base image
we chose (`box-cutter/ubuntu1404-docker`) has also an image for Parallels and Vagrant can choose whichever is installed.

At the beginning of the Vagrantfile right below

    Vagrant.configure(2) do |config|

add the configuration to prefer Parallels but fallback to VirtualBox,
if Vagrant can't find Parallels, like this:

    Vagrant.configure(2) do |config|
      config.vm.provider "parallels"
      config.vm.provider "virtualbox"

And now, add the cool features for Parallels:

    config.vm.provider "parallels" do |prl|
      prl.use_linked_clone = true
      prl.update_guest_tools = true
      # Customize the amount of memory on the VM:
      prl.memory = "256"
    end

We're limiting the Parallels VM also to only 256MB of memory. But additionally, we use linked clones.
This means, instead of copying the base image, every time we launch a new VM, the new VM will be just
a clone of the base image. This makes starting up fresh Vagrant VMs in Parallels much faster and consumes less storage space.

# Why not just using [boot2docker](https://github.com/boot2docker/boot2docker)?

First of all, the [Windows installer is deprecated](https://github.com/boot2docker/windows-installer/releases)
and replaced with the [Docker Toolbox](https://www.docker.com/products/docker-toolbox), but ok, than the question changes just slightly.

Secondly, I started using Vagrant + Docker before boot2docker became popular. That's great but doesn't answer, why I didn't change.

Thirdly, usually, you don't just need a Docker image, but you also want to have scripts which set up some configuration or similar.
This is from my experience much easier to be done inside a Vagrantfile. So I'd use it even in Linux sometimes.

# HELP - it says docker isn't installed, WTF!?

Calm down, breath. I also had this error, don't know when anymore but there is a simple fix.
Add this line into the `config.vm.provision` section, before you try to use docker:

    type docker > /dev/null 2>&1 || { wget -qO- https://get.docker.com/ | sh; }

This checks if the command `docker` is available and if not, it installs it.

# Final version

And this is my final Vagrantfile, enjoy:

<script src="https://gist.github.com/pgrm/6cb9d4566f3f383a6e90.js"></script>
