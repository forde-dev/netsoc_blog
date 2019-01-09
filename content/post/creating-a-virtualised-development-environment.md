---
title: "Creating a Virtualised Development Environment"
date: 2018-11-30T16:52:29Z
draft: false
tags: ["tutorials"]
author: Evan Smith
---

# Creating A Virtualised Development Environment

### Vagrant and VirtualBox

For our development environment, we’re going to want a way to easily simulate an ubuntu/linux server as well as an easy way to interact with it when we need to make changes. That’s where Vagrant and VirtualBox come in 😉

#### VirtualBox

VirtualBox is a cross-platform virtualization application. It allows us to simulate having an operating system without having to use a physical computer for it. It’s also handy to simulate specific types of environments while also keeping your main machine secure and uncluttered. We’ll be using VirtualBox as our virtualization component in this tutorial.

You can download VirtualBox for your platform at https://www.VirtualBox.org/wiki/Downloads

#### Vagrant

Vagrant is a tool that sits on top of VirtualBox (or VMWare or AWS) and gives us an easy way to provision and interact with virtualized Operating Systems without having to manually configure them ourselves (well, you won’t anyway). It’s a handy way to use pre-built “boxes” to quickly set up and destroy dev environments.

You can download Vagrant for your platform at http://www.vagrantup.com/downloads

### Using Our Box

We’ve handily made a pre-built box for you to get up and running yourself. After you’ve installed virtualbox and vagrant, follow the below instructions to get our box up and running for you.

First, open Terminal (OSX or Linux) or Command Prompt (Windows) and `cd` (change directory) into a folder you’ve created for your development environment. Then execute the following commands:

```bash
# Adds the box permanently to our collection as well as downloads it
vagrant box add Netsoc https://netsoc.co/netsoc_boxes.json

# Prepares the current directory with a Vagrantfile (a config file)
vagrant init Netsoc

# Launches the Virtual Machine
vagrant up

# Shutdown the Virtual Machine
vagrant destroy
```

It may take a while for the file to download and then again when you launch the machine but now there’s only one last step to our vagrant setup and that’s the custom Vagrantfile. Simply open the Vagrantfile in your directory, replace the content with the content below and you’re good to go!

There are comments at every step in the code below but if you need a further explanation, you can find out more from the [vagrant documentation](https://www.vagrantup.com/docs/vagrantfile/index.html)

```ruby
project_name = "netsoc"
ip_address = "172.22.22.22"

# Begin our configuration using V2 of the API
Vagrant.configure(2) do |config|
  
  config.vm.box = "Netsoc"

  # Configuration for our virtualisation provider
  config.vm.provider "VirtualBox" do |vb|
     # Memory (RAM) capped at 1024mb
    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end

  # Give our new VM a fake IP Address and domain name
  # To utilise this, add the following to your /etc/hosts file
  #   172.22.22.22 netsoc.dev
  config.vm.define project_name do |node|
    node.vm.hostname = project_name + ".dev"
    node.vm.network :private_network, ip: ip_address
  end

  # Sync the containing folder to the web directory of the VM
  #   The sync will persist as you edit files, you won't have
  #   to destroy and re-up the VM each time you make a change
  #   
  config.vm.synced_folder "./", "/var/www/html", :owner=&gt; 'www-data', :group=&gt;'www-data'
end
```

If you’re on Linux/OSXc you can now SSH into your box with `vagrant ssh`. If you’re on windows however, there are two options:

1. If you have Git installed (which you really should and can download here), you simply issue set PATH=%PATH%;C:\Program Files (x86)\Git\bin in the CMD you have open. For reference, this fix comes from this Github issue
2.  Otherwise, you can use Pageant and Putty. I’m not going to cover it here but there is a guide available

### What’s In The Box?

Our box is made ideally for web development and sysadmin tutorials we’ll be doing throughout the year. Installed in the box is:

1. Apache2
2. PHP 5.6
3. MySQL
4. Composer & LAravel

### How We Made Our Box

Our box was made in an OSX environment.

```bash
mkdir vagrant_test
cd vagrant_test

vagrant box add http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box --name=Ubuntu14.04
vagrant init Ubuntu14.04
vagrant up

# Log in to the box
vagrant ssh

	sudo apt-get update
	sudo apt-get install lamp-server^

	## Chose mysql password: root

	#Install PHP 5.6 over the 5.5.9 installed by default
	sudo add-apt-repository ppa:ondrej/php5-5.6
	sudo apt-get update
	sudo apt-get install python-software-properties
	sudo apt-get install php5

	#Install composer
	curl -sS https://getcomposer.org/installer | php
	sudo mv composer.phar /usr/local/bin/composer

	#Install laravel
	composer global require "laravel/installer=~1.1"

	echo 'export PATH="$PATH:~/.composer/vendor/bin"' &gt;&gt; .profile
	. .profile
	# We won't create a laravel blog here as this is supposed to be an empty box.
	# If you want to test laravel, type "laravel new blog" and check for a
	# "blog" directory.

	#Allow .htaccess in apache
	cd /etc/apache2
	sudo vi apache2.conf

	# Look for the following: 
	# &lt;Directory /var/www/&gt;
	#         Options Indexes FollowSymLinks
	#         AllowOverride none
	#         Require all granted
	# &lt;/Directory&gt;
	#
	# and change to:
	# &lt;Directory /var/www/&gt;
	#	        Options Indexes FollowSymLinks
	#	        AllowOverride All
	#	        Require all granted
	# &lt;/Directory&gt;

	sudo service apache2 restart

	# Enable Mod_Rewrite for rewriting URLs
    sudo a2enmod rewrite
    sudo service apache2 restart

	# This may or may not be required as I've heard mixed opinions but 
	# we had to re-install the Virtualbox guest additions again otherwise
	# SSH would fail on vagrant up in the re-packaged box
	cd ~
	sudo apt-get install linux-headers-generic build-essential dkms
	sudo apt-get -y -q purge virtualbox-guest-dkms virtualbox-guest-utils virtualbox-guest-x11
	wget http://dlc-cdn.sun.com/virtualbox/4.3.8/VBoxGuestAdditions_4.3.8.iso
	sudo mkdir /media/VBoxGuestAdditions
	sudo mount -o loop,ro VBoxGuestAdditions_4.3.8.iso /media/VBoxGuestAdditions
	sudo sh /media/VBoxGuestAdditions/VBoxLinuxAdditions.run
	rm VBoxGuestAdditions_4.3.8.iso
	sudo umount /media/VBoxGuestAdditions
	sudo rmdir /media/VBoxGuestAdditions

	# Exit the VM
	logout
	
# Packages a running vagrant box
vagrant package --output Netsoc.box
	
# Leave the current directory and create a blank one
# just to be sure we're starting anew
cd ../
mkdir netsoc_test
cd netsoc_test

# Add the box and launch it
vagrant box add Netsoc.box --name=Netsoc
vagrant init Netsoc
vagrant up
vagrant ssh
	#Success
vagrant destroy
```

### Debugging

In the event that you begin to get an “authentication failure” when vagrant upping, you can still ssh into the box with the user “vagrant” and password “vagrant using the following command:

```bash
ssh vagrant@localhost -p 2222
```

