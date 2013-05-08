---
layout: post
title: SSH authentication with Mosh
---

{{ page.title }}
================

<p class="meta">8 May 2013</p>

I've been using [Mosh](http://mosh.mit.edu/) for a while to work on remote
development virtual machines.  I do a lot of work on the train while commuting,
and being able to have a persistent session which I can access wherever I am is
great.  The reduced latency and notifications of unsent content that mosh
provides are also very useful features.

The only significant downside has been that Mosh doesn't support authentication
forwarding.  This means that to perform actions such as git pulls over SSH, I
need to log into the machine separately over raw SSH, and perform the action in
a separate terminal.

I was just shown a [blog
post](http://blog.codersbase.com/2012/03/tmux-ssh-agent.html) which solves this
problem for tmux by pointing the `SSH_AUTH_SOCK` environment variable on the
remote machine at a symlink, and updating that symlink on each reconnection to
point to the real authentication socket.  I'm now using a variant on this
approach with mosh, as follows:

First, as suggested for tmux, I have added a `$HOME/.ssh/rc` file on the remote
server with the following contents:

    #!/bin/bash
    if test "$SSH_AUTH_SOCK" ; then
        ln -sf $SSH_AUTH_SOCK ~/.ssh/ssh_auth_sock
    fi

This causes a symlink at `$HOME/.ssh/ssh_auth_sock` to be updated to point to
the SSH auth socket on each connection to the machine.  Note that keeping this
symlink inside the .ssh directory is a good idea, since that directory should
always have nice restrictive permissions to stop any other user on your system
accessing it.

Then, I added a line to my `$HOME/.bashrc` file to set the SSH_AUTH_SOCK
environment variable on login to point to the symlink.  This is run on login,
after the .ssh/rc file:

    export SSH_AUTH_SOCK=$HOME/.ssh/ssh_auth_sock

As a result, in my mosh session, the SSH authentication socket used comes from
whichever ssh connection I last made to the machine.

Finally, I run a process in the background on my laptop, which connects to the
remote server over plain SSH (and also makes a few useful SSH tunnels, while
it's at it).  Whenever this process loses its connection, it sleeps a little
bit and then tries to reconnect.  As a result, I can always access the SSH
authentication from my laptop in my mosh session if there's a reasonable
network connection between the two, without needing to leave my comfortable
mosh session.
