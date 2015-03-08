---
layout: post
title:  "Your development environment"
date:   2015-02-23 23:00:00
---
When i wanted to encourage some people to join [YAMEM][yamem-site] dev team big problem popped out. How to setup dev machine that it fulfill project requirements. At that time i said "You need to install this, those and that software" which of course is mistake by default. The reason of this is fairly simple. They're using OSX/Windows, me Ubuntu.

<!-- more -->

## Virtual machine

Ideal solution is to use virtual machine that can be spin up on any computer and be ready to use for a developer. First tools that comes to mind is [VirtualBox][virtual-box] in which you can setup your virtual machine and distribute it to other users. That's great. So every one uses the same machine, tools, versions, etc.

But some problems might (for sure) occur. What if you want to use new version of something? Maybe tune configuration. Perhaps want to throw in some new tools. Or you just messed configuration and everything is broken.

In those cases you might want to remove broken/obsolete VM, grab new one, spin it up and do your work. That would be great but ... it's repeatable work. And what developers like the most? Automate things :)

## Vagrant

There is [Vagrant][vagrant-up] project which solves this problem. Bottom line is that you create virtual machine and configure it. Then you pack virtual machine to `.box` file and share it with other.

Basic Vagrant configuration workflow

* initialize Vagrant with `vagrant init`
* configure `Vagrantfile`
* install all necessary tools on VM
* pack VM into a single file with `vagrant package --base [filename]`
* include link to `.box` file in `Vagrantfile`

And here goes basic Vagrant usage workflow:

* `vagrant up`
* `vagrant ssh`
* do you work
* `vagrant halt` or `vagrant destroy`

Command `vagrant up` basically says: 'Hey bring this VM to life and set it up!'. It means Vagrant will try to:

* find VM in imported boxes
* if fail and if link to box is provided it'll download and import box from provided link
* Vagrant version later than 1.5 will try to find box in [own library][vagrant-atlas]
* make a copy of that box for project and boot it up

Rest of commands is basically self explanatory but `vagrant destroy`. Destroy means delete copy of box that is used not the imported one.

## One step fruther

There is one more thing. If your project is hosted on VPS you certainly have specific configuration for this server. Good practice is to develop on envoirnment configured as close as possible to prodction/live configuration. That's because if you upgrade some tools, libs or apply security patches you don't want to be surprised when something goes wrong.

There are other tools (of course they are) that helps to manage those server configurations. For example [chef][chef-site], [puppet][puppet-site], [ansible][ansible-site] and many other. And of course you can apply them to configure you Vagrant box as your production server.

I'll cover at least one of them so stay tuned.

{% include links_library.md %}
