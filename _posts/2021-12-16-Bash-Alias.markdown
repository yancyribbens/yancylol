---
layout: post
title:  "Bash Aliases"
date:   2021-12-16 14:02:00 +0100
categories: rust
---

I've been working a lot in Rust lately (more on that later), and this requires me to invoke rust jail a lot (like every time I turn on the computer). I wanted to streamline my workflow when I get going so I have less to remember and think about (because thinking hurts).  To do this, I have a few bash aliases in mind which makes startup quicker, and while I find aliases helpful, every time I go to create aliases it turns into a journey.

The question is, which of the many places is best to create bash aliases.  The "man bash" page lists files: `/usr/local/etc/profile`, `~/.bash_profile`, `~/.bash_login`, `~/.profile` and `~/.bashrc` as different places that are read.  After a bit of cyber sleuthing I found advice that bashrc is for non-login shells and other files such as `bash_profile` and `profile` are for login shells.  What that means in terms of aliases is that it's preferable to place bash aliases in a non-login shell, which are read every time a new bash shell is opened.  A login shell on the other hand seems to be a higher level shell which reads a profile that is system wide (during login), and so the aliases placed here may become available in the future to any shell, not only bash. 

I created a file called `~/.bash_aliases` and placed the following:

{% highlight nix %}
function start() {
  sudo /etc/rc.d/jail onestart "$@" ;
}

function restart() {
  sudo /etc/rc.d/jail onerestart "$@" ;
}

function stop() {
  sudo /etc/rc.d/jail onestop "$@" ;
}

function bash() {
  sudo jexec $1 /usr/local/bin/bash \-login;
}
{% endhighlight %}

next the following is added to ~/.bashrc


{% highlight nix %}
if [ -e $HOME/.bash_aliases ]; then
  source $HOME/.bash_aliases
fi
{% endhighlight %}

I then source .bashrc which brings me to a new problem:	

Username is not in the sudoers file. This incident will be reported:

Hopefully the police aren't waiting for me outside.  The rust environment must be run by root (or sudo) but it appears my user is not allowed.  Using the root account and the visudo utility to edit the sudoers file, I appended the following to the end of the file:
      </p>

{% highlight nix %}
wintermute ALL=(ALL) ALL
{% endhighlight %}

Well, that seemed to solve my problem, and when I run Konsole, my current default terminal, I can now run "rust" and "rust-shell" to start a rust environment and invoke a shell.   However, a new wrinkle prevents  the aliases from working when I invoke "tmux" from inside konsole, and then type rust.  It turns out the tmux invokes an interactive login shell instead of an interactive non-login shell and so bashrc is bypassed.  Back to detective mode and I found a suggestion to place the following into ~/.bash_profile

{% highlight nix %}
if [ -n "$BASH_VERSION" -a -n "$PS1" ]; then
  # include .bashrc if it exists
  if [ -f "$HOME/.bashrc" ]; then
    "$HOME/.bashrc"
  fi
fi
{% endhighlight %}

Everything seems to work now on startup, and when invoking tmux, ".bash_profile" is read and if the shell is a bash shell and if the shell is interactive then .bashrc is loaded and consequently so are the aliases.  Whew, that was a lot of work to save a small amount of typing every time, but the time savings add up.  The next challenge is that each time I start the jail, the IP address needs to be changed in /etc/jails.conf to the current interface address.  To be continued.
