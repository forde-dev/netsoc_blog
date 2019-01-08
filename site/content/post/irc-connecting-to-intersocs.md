---
title: "IRC - Connecting to Intersocs"
date: 2018-11-30T16:49:50Z
draft: false
tags: ["tutorials"]
author: Evan Smith
---

# IRC – Connecting To #Intersocs

### User Accounts

If you do not already have a user account for UCC Netsoc’s services, sign up at https://admin.netsoc.co/register

**You will need a user account before continuing with this document.**

### Quickstart

If this is your first time, please read the entirety of the document as we do not provide support as of yet.

1. SSH into Leela with your user credentials: `ssh USERNAME@leela.netsoc.co`
2. Access your Weechat IRC screen: `irc`
3. `/connect netsoc.co`
4. `/join uccnetsoc`
5. `/join intersocs`
6. When you’re finished, hit `ctrl` + `a` + `d`

### What is IRC?

IRC (Internet Relay Chat) is a chat protocol that allows us to communicate with all the networking/computer societies on the **Intersocs** network.

#### What The Fuck Is Intersocs?

Intersocs is a network of IRC “peers” that facilitates communication with the following societies:

* Redbrick (DCU Netsoc)
* TCD Netsoc
* DUCSS (Dublin University Computer Science Society)
* Skynet (UL Netsoc)
* DIT Netsoc
* UCD Netsoc (Coming Soon…)
* NCI Netsoc (Coming Soon…)
* NUIG CompSoc (Coming Soon…)

### Logging in (SSH)

The first thing we’re going to need to do, is actually log into Leela (our primary user server). To do this, we’ll use **Secure Shell** (SSH).

#### Mac OSX / Linux

SSH clients come preinstalled on all versions of Mac OSX and Linux. To start using it, open up your terminal/console/command prompt of some kind, replace `USERNAME` with your username and execute the following command:
 
```console
ssh USERNAME@leela.netsoc.co
```

You’ll then be prompted for your password.

#### Windows

You’ll need to install some kind of SSH client for windows. We recommend using [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

To use, PuTTY, I’d recommend this [tutorial](https://mediatemple.net/community/products/dv/204404604/using-ssh-in-putty-).

You’ll need the following details:
 
```
hostname: 	leela.netsoc.co
username: 	YOUR_USERNAME
port: 		22
```

### Screen

The `screen` command is what’s known as a \texit{terminal multiplexer} (or “mux”). What it does is it creates multiple windows for you to run different commands or leave commands running. It’s useful if you want to leave something connected but still want to be able to log out.

We’re going to use screen to keep a permanently connected irc client running for whenever you log in.

#### Our `irc` command

When you log in, you’re automatically given a `.profile` file with some predefined commands. One of those commands is `irc`.

What happens is that when you log in, we automagically create a screen for you (if it isn’t running already) and we alias} “\inlinecode{irc” to attach you to that screen.

#### Manually Creating A Screen

You can also manually create screens yourself. Below is part of the example `.profile` we give everyone. You only need to concern yourself with lines 2 and 5.

```bash
if ! screen -list | grep -q "irc"; then
        screen -dmS irc bash -c 'weechat'
fi

alias irc="screen -r irc"
```

`screen -dmS irc bash -c 'weechat'`

Here we’re:

1. creating a screen
2. telling it to run in the background
3. naming it “irc”
4. executing the command `bash -c 'weechat'` – you can leave this blank and it will run a bash shell by default

`alias irc="screen -r irc"`

This line simply aliases the command `irc` to command `screen -r irc`. We can attach ourselves to any named screen using the syntax `screen -r NAME`

#### Okay… I want to leave. Now what?

To detach from any screen, simply hit `ctrl` + `a` + `d` and you’ll be returned to your original terminal.

### IRC

This is it, the bit you’ve all been waiting for.

Once you’ve connected to your IRC screen (that’s running Weechat), you will need to connect to our netsoc server with the following command:
 
```console
/connect netsoc.co
```

Congrats, you’re now connected! Start out by typing `/join uccnetsoc` and `/join intersocs`
When you’re done

Simply `ctrl` + `a` + `d` to detach from your screen and type `logout` to leave the server.
