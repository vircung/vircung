---
layout: post
title:  "Rails Sandbox"
cover: https://static.pexels.com/photos/2693/call-box-phone-box-phones-public.jpg
---

In previous post i scratched the topic of managing and sharing your development environment. By using Vagrant to manage virtual machines and one of provision tools you can setup configurable environment. Furthermore you can setup your production machine to match this configuration. Lets roll up our sleeves and set it up.

<!-- more -->

## Vagrant setup

You need to install few things: [Vagrant][vagrant-up], VM manager (i'll choose [VirtualBox][virtual-box]) and [ruby][ruby-site]. Also as a base linux distribution i'll use Ubuntu.

First step is to prepare Vagrant configuration so execute `vagrant init` command. This will generate sample `Vagrantfile` with lots of helpfull comments, for now you can delete them to keep configuration clean and tidy. As a box name use `ubuntu/trusty64`.

{% gist 9d535d659d116aec00e6/1c11e0671c057716ce150a1b3ad374527b2f8a3e Vagrantfile %}

Now each time when you execute `vagrant up` vagrant will search for box with this name. If doesn't find it in imported boxes will search his [online catalog][vagrant-atlas] and download it for you. You can also safely `vagrant destroy` if needed (for sure we will use it later).

## Provisioning

Vagrant has built in functionality to [provision][vagrant-provision] box while creating it. What it means is that you can execute custom scripts that will install and configure software for you. There are some tools that support this kind of operations which i mentioned in previous post. For now i will use [chef solo][vagrant-chef-solo] provisioner.

In short Chef provisioning uses sets of instructions (called recipes) to setup virtual machine. Recipes can be configured with attributes use templates and be separated in several files. All of those can be combined into Cookbook. There are many configured cookbooks and can be found on [Supermarket][chef-supermarket]. Some of them are fine tuned and can be found on [GitHub][github-site].

{% gist 9d535d659d116aec00e6/1c11e0671c057716ce150a1b3ad374527b2f8a3e Cheffile %}

To easily use and manage existing cookbooks install [librarian-chef][librarian-chef] ruby gem `gem install librarian-chef` and initialize it `librarian-chef init` in folder with `Vagrantfile`. Chef configuration file `Cheffile` will contain commented out examples delete them to keep it clean and tidy.

You can group and tune cookbooks into roles. Most often they are grouped into functional groups, for example if your environment is split into 2 VMs (database server and application server) you will end up with 2 roles: database and application. Let's create one role to keep Chef configuration out of `Vagrantfile`.

Create folder `roles` and inside of it ruby script `base.rb`. Role scripts needs to have name so add `name 'base'`. We will add more to it when we add some cookbooks.

{% gist 9d535d659d116aec00e6/78740962f915f44084a196434bebbea39fbd5310 base.rb %}

Now we need to tell Vagrant where is should search for cookbooks and roles. Also add this role to actual vagrant box.

{% gist 9d535d659d116aec00e6/78740962f915f44084a196434bebbea39fbd5310 Vagrantfile %}

## Cooking ruby

Now lets add first cookbook to install [RVM][rvm-site] and configure it to install newest ruby version (2.2.0). Add `cookbook 'rvm', '~> 0.9.2'` to `Cheffile` and execute `librarian-chef install`. Cookbook for rvm and all it's dependencies will be places in `cookbooks` folder which will be searched by Vagrant provisioning.

{% gist 9d535d659d116aec00e6/dff06e9ebb111ab55d5a2fbfffa2b96b85c6d7cb Cheffile %}

Next step is to adjust `base` role to install rvm and ruby. To install rvm system-wide accordingly to [README][chef-rvm] we need to add `recipe[rvm::system]` recipe to run list.

Now lets adjust attributes to install ruby 2.2.0, use it as a default and  gem `bundler`. Also to give user `vagrant` ability to install gems we need to add it to `rvm` group.

{% gist 9d535d659d116aec00e6/dff06e9ebb111ab55d5a2fbfffa2b96b85c6d7cb base.rb %}

Now lets execute `vagrant reload` to refresh synchronized folders and `vagrant provision` to make use of installed cookbooks and set role. It can take a few minutes to complete. When finished you can `vagrant ssh` into machine and check version `ruby -v` and installation folder of ruby `which ruby`.

Every time when you execute `vagrant destroy` and `vagrant up` you will end up with VM configure exactly same way.

## Cooking postgresql

As a database backend PostgreSQL is used frequently if application is either self hosted on VPS or on Heroku. In second case PostgreSQL if forced by hosting provider. So having it configured in development enviornment is helpful. Let's add required cookbook and adjust our role.

{% gist 9d535d659d116aec00e6/8253a79ca628cabb6f825326d87da4d14a39b220 Cheffile %}

Don't forget to execute `librarian-chef install` to actually install new cookbook.

{% gist 9d535d659d116aec00e6/8253a79ca628cabb6f825326d87da4d14a39b220 base.rb %}

If we are using standalone version of chef (other option is that chef recipies are managed by dedicated server) there is a need to setup password for `postgres` user. Another catch is that to allow `postgres` user to be authenticated by password. That is why we set `pg_hba` variables which are reflected in configuration with same name. I took value of this attribute from original recipe and changed authentication method from `ident` to `md5` for `postgres` user.

## Last touches

Last things to setup in vagrant is to expose port that is used by rails to host machine. Current folder is mounted in `/vagrant` path by default. If project is kept in different folder than folder with Vagrantfile you need to explicit setup it to be mounted in VM. Let's do this and we will be able to create new project!

{% gist 9d535d659d116aec00e6/13572e1ae1d77e66b26842b2a305063260d82f23 Vagrantfile %}

All you need now is to `vagrant up` then `vagrant ssh` and `cd /path/to/project/in/vm` to be in your development sandbox. Your default machine won't be cluttered with many per project configurations. You can easily experiment with different setups and event split your application into many dedicated servers. Most important it is OS agnostic, easily replicable, configuration consistent and can be shared with everyone.

Please be warned to keep password for production database safe and secure. This Chef configuration is written for development environment not production.

{% include links_library.md %}

[virtual-box]:https://www.virtualbox.org/
[vagrant-up]:https://www.vagrantup.com/
[vagrant-atlas]:https://atlas.hashicorp.com/boxes/search
