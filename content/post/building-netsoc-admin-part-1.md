---
title: "Building Netsoc Admin Part 1"
date: 2018-12-01T15:43:59Z
draft: false
tags: ["dev"]
author: Evan Smith
---

# Building Netsoc Admin (#1) – Development Tools

### Introduction

Last September we ([UCC Netsoc](http://netsoc.co/)) installed two servers and dedicated one of them specifically to servicing our members and their applications. It allows people to SSH in and run PHP, python and JS-based apps off each user’s subdomain. We wanted people to be able to manage their own info, databases and could access automated backups. After a month of looking at web panels and various solutions, it was apparent that what we wanted didn’t exist in a pre-built package unfortunately. The software we found was ugly, lacking and, worst of all, annoyingly complex to change.

So here we are, rolling our own and re-inventing a wheel probably countless others have investigated/built. However, I hope that you can learn from what we’ve built (and maybe even extend it). This is a series of blogs detailing my process of building a Laravel WebAdmin for our Leela server.

Here’s the link to our github repo. If you spot any errors or have any suggestions, please submit it to our issue tracker – we’d really appreciate it 🙂

### My Development Workflow

In this post, I want to walk you through my day-to-day development and the technologies I use during development. The main idea behind everything I talk about here is that everything should be local. If I have to board a submarine to the arctic, I should still be able to work. Admittedly, there are 3 lines of code that contradict this because I link to a CDN for MaterializeCSS, FontAwesome and jQuery. However, I have my own copies of these files locally should I ever need to swap them in for the CDN files. Regardless, develop local – it’s better for the economy.

#### 1) Vagrant

Vagrant is a virtualisation manager for VirtualBox. It creates easy-to-modify virtual machines based on a Vagrantfile in the root project folder. Netsoc has a prebuilt box that we use for PHP development so the Vagrantfile in use here is based off that.

First thing’s first though, I’m going to be running my project off the IP `172.22.22.25`. `172.16.0.0/12` is a private range of IP addresses so there should be no issue in interfering with any normal traffic we’d be using our machine for. As such, I create an entry in my hosts file (`/etc/hosts` for Mac and Linux) of the form:

```console
172.22.22.25 netsocadmin.dev
```

Now I can use the address `netsocadmin.dev` to access the web server. Next, I’ll want to initialize my project with a Vagrantfile so it can provision the virtual machine automatically whenever I need it. For that, I use the following:

```ruby
project_name = "netsocadmin"
ip_address = "172.22.22.25"

# Begin our configuration using V2 of the API
Vagrant.configure(2) do |config|
  
  config.vm.box = "Netsoc"
  config.vm.box_url = "http://files.netsoc.co/f/2e06d99781/?dl=1"
  config.vm.box_check_update = true

  # Configuration for our virtualisation provider
  config.vm.provider "VirtualBox" do |vb|
     # Memory (RAM) capped at 1024mb
    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end

  # Give our new VM a fake IP Address and domain name
  # To utilise this, add the following to your /etc/hosts file
  #   172.22.22.25 netsocadmin.dev
  config.vm.define project_name do |node|
    node.vm.hostname = project_name + ".dev"
    node.vm.network :private_network, ip: ip_address
  end

  # Sync the containing folder to the web directory of the VM
  #   The sync will persist as you edit files, you won't have
  #   to destroy and re-up the VM each time you make a change
  #   
  config.vm.synced_folder "./", "/var/www", :owner=&gt; 'www-data', :group=&gt;'www-data'
  config.vm.synced_folder "./public", "/var/www/html", :owner=&gt; 'www-data', :group=&gt;'www-data'

  config.vm.provision "shell", inline: &lt;&lt;-SHELL

  	sudo apt-get update 
  	sudo apt-get install -y php5-ldap
  	sudo service apache2 restart

    # Create our database and give root all permissions
    mysql -uroot -proot -e "CREATE DATABASE IF NOT EXISTS #{project_name};"
    mysql -uroot -proot -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root';"
    sudo service mysql restart
    
    #Create swap space for composer's operations
    sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
    sudo /sbin/mkswap /var/swap.1
    sudo /sbin/swapon /var/swap.1

    # Update laravel and create all the DB tables
    cd /var/www/
    sudo composer update
    sudo php artisan migrate
    sudo php artisan db:seed
  SHELL
end
```
When that’s all setup, it’s time to launch the virtual machine and get the server running.

* To launch a server, use `vagrant up`.

* To put a server to sleep, use `vagrant halt`.

* To shutdown the server (and free up the space on your computer), use `vagrant destroy`.

### 2) Docker (where necessary)

Docker is a “container” manager. Containers are essentially a box that wrap up a piece of software in a complete filesystem that contains everything it needs to run: code, runtime, system tools, system libraries – anything you can install on a server.

In the case of this project, we needed to be able to test with an LDAP server. For this, the best option was running the LDAP instance from a docker container so we weren’t messing with in-production systems and could fiddle and fool all we wanted. (Just a small note: I’m omitting the fact that I used a backup of our current LDAP database and mounted them as volumes to the following docker container. This was only necessary because we had data already and not working from a blank system). To startup our docker container, I used the following:

```console	
docker run -p 389:389 --name ldap_server -e LDAP_DOMAIN="netsoc.co" -e LDAP_ORGANISATION="UCC Netsoc" -e LDAP_ADMIN_PASSWORD="password" -d osixia/openldap
```

### 3) Laravel

Laravel is a PHP MVC Framework I personally like to use because of its integration with the command line, ease-of-use, model relationships, queues and scheduler. There are many frameworks out there that have different pros and cons and I encourage you to find one in a language you enjoy to help black-box some of the boiler plate necessary for starting a new project (EG: user registration or page routing).

Laravel uses Composer so once that’s installed, I can create a blank project to work from.

```console
laravel new blog
```
	
We’ll also need some basic scaffolding for the user registration, so I’m also going to run

```console
php artisan make:auth
```

### 4) Sublime (Text Editing)

Again, Sublime Text is a personal preference of mine. Combined with Package Control, Sublime is a comfortable and efficient editor for any language. I’ve used it for python, perl, ruby, bash, PHP, HTML, CSS and for taking notes in college. It’s versatile and beautiful at its core but here are some plugins I find especially useful.

#### DocBlockr

This a plugin for better code commenting/documentation. It provides a couple of neat extras to make commenting much easier but its core principle really lies within auto-generating javadoc/phpdoc-style comments for functions after typing `/**` and pressing `enter`

#### Emmet

Emmet lets you type out a hierarchy of elements in the style of CSS declarations and then auto-expands them into HTML code. For example `(li>a.anchor-class)` creates an `li` around an `a` tag with the class `anchor-class`

#### SFTP

SFTP is SFTP. It will let you sync up files to a remote server whenever you save or you can choose to sync up/down changed files manually. There are a couple other really cute features too and I really love this plugin for working on remote servers for client work/hotfixes.

### 5) MaterializeCSS (CSS/HTML)

Materialize is a CSS and HTML framework that makes building templates and pages incredibly nice and intuitive. Most of the manipulation comes through applying classes to elements and stacking the HTML nicely. It implements a lot of the concepts behind Google’s Material Design concept which they implement in Android. This is, in my opinion, the best CSS/HTML framework out there at the moment. With minimal effort, the user experience is responsive both on a device and user level. It’s a joy to work with and a joy to use.

### 6) LESS (CSS)

Materialize is great and covers most of the styling I’d need but, for everything else, there’s LESS. Less is a pre-processor for CSS that allows me to write nested CSS declarations, fiddle with variables and do computations which it will then expand into proper, valid CSS for the browser to interpret. It makes writing and understanding CSS a lot easier with minimal effort. It also makes it incredibly easy to define app-specific colours and change them at a whim. No longer do I have to perform a find and replace only to find someone thought they should roll their own shade of blue into my pristine web app (damn you! DAMN YOU!).

### 7) Gulp

Gulp is a compiler for my javascript and less. I know that sounds weird but hear me out, okay? By running gulp watch it will create a browser window and watch my files for changes. Then, whenever a change is made to a relevant file, it will process my less, put all my javascript together in one file, minify/squash it all and then reload the browser window. This way, I can lay out clean and neat files while still sending the most compact version to the browser.

All it requires is the below gulpfile to work:

```javascript
var elixir = require('laravel-elixir');

elixir(function(mix) {
	// Mix all of our javascript and less files
    mix.less('app.less');
    mix.scriptsIn("resources/assets/js/app", "public/js/app.js");
    mix.scriptsIn("resources/assets/js/wordpress", "public/js/wordpress.js");
});

// This is for syncing the browser 
require('laravel-elixir-browser-sync');

// Any time a file is changed in the below folders,
// reload the browser window
elixir(function(mix) {
	mix.browserSync([
	    'public/**/*',
	    'resources/**/*'
		], {
		proxy: 'netsocadmin.dev',
		reloadDelay: 500
	});
});
```
	
### 8) Github (Version Control)

Version control is, arguably, the most important part of this entire workflow. Source/Version control means that there’s a clear backup of everything I do as well as giving other developers an insight into my thought process when investigating files later. It’s amazing for collaborating with many people at the same time and still maintaining usable code. Github specifically also offers us a single place to distribute our code and allow people to submit issues/suggestions (which you should totally do)

### There are many like it but this one is mine.

I hope you’ve learned something from all of the above, even if it’s just a quirky “huh, I should try that”. Workflows differ greatly for different people and different languages and frameworks (for instance, python’s Django-ers probably won’t use vagrant to server their code) but this is what works for me and it gives me 3 main benefits:

1. Rapid development
2. Security and backups
3. Reliability when it comes to deployment

### What are some things you’d like to incorporate in your workflow?

I’ve gazed longingly at unit-tests for so very long but have yet to cross the nightclub and ask for a dance. I understand the benefits of writing and adhering to unit tests as you develop but unfortunately my current rate at which I go from development to production is just too quick and small-time for me to slow things down with proper unit-testing but, I hope to change my spots some day and take on that 4th benefit: guaranteed code quality.

