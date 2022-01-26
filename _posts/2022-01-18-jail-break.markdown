---
layout: post
title:  "Jail break"
date:   2022-01-18 14:02:00 +0100
categories: freebsd 
---

Yesterday I finished an early version of a new utility I call Crate-r, although that's not what this post is about.  I wanted to test the utility in a new jail, and made some mistakes and encountered a persistant problem.  When trying to delete the Jail using the root account, I get the following error.

{% highlight nix %}
rm: director_name: Operation not permitted
{% endhighlight %}

When a Jail is created, there is a system immutable flag that is set which prevents even the root account from deleting said files.  To set (and unset) the immutable flag, the venerable chflags system utility is useful.  See "man chflags" not to be confused with chflags man.  Jails are a combination of a number of files, so the recursive flag is also useful.

Recursively clear all flags on files and directories contained within the jailname directory hierarchy (or as I like to call it a jail break).

{% highlight nix %}
chflags -R 0 ./jailname
{% endhighlight %}
