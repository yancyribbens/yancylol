---
layout: post
title:  "Running Apache in Jail"
date:   2021-11-17 14:02:00 +0100
categories: freebsd
---

Recently I've been using FreeBSD for development, in part because I really like the experience of developing inside of jails.  No, I'm not delinquent for tax evasion, what I mean is that FreeBSD provides something called a <a href="https://docs.freebsd.org/en/books/handbook/jails/"> Jail </a>, which I have found is a happy medium between a VM and a Linux container.  Jails, like containers use the host kernel, however, are less ephemeral than containers.  Here's how I created a container to run Apache for developing this blog.  Note in the future I want to use a different web server, maybe something I develop in Rust.

FreeBSD Version: 13.0-RELEASE.

install a new jail to /root/jails/apache
{% highlight shell %}
bsdinstall jail /root/jails/apache
{% endhighlight %}

edit jail.conf
{% highlight shell %}
vim /etc/jail.conf
{% endhighlight %}

paste the following
{% highlight nix %}
apache {
  host.hostname = phobos;
  ip4.addr = 192.168.0.1;                    # IP address of the jail (the internet facing IP)
  path = "/root/jails/apache";               # Path to the jail
  mount.devfs;                               # Mount devfs inside the jail
  exec.start = "/bin/sh /etc/rc";            # Start command
  exec.stop = "/bin/sh /etc/rc.shutdown";    # Stop command
}
{% endhighlight %}


start the jail called apache
{% highlight shell %}
/etc/rc.d/jail onestart apache
{% endhighlight %}

invoke the default shell
{% highlight shell %}
sudo jexec apache /bin/sh
{% endhighlight %}

update, upgrade and install bash
{% highlight shell %}
pkg update; pkg upgrade; pkg ins bash
{% endhighlight %}

exit current shell
{% highlight shell %}
exit
{% endhighlight %}

We can now use bash!
{% highlight shell %}
jexec apache /usr/local/bin/bash
{% endhighlight %}

I needed to run the following command on the host (not the jail) to give permissions to use raw_sockets.  Without this, an ominous error message is displayed when in the Jail and using the interface, for example to ping.
edit: This is only needed for certain network capabilities such as ping
{% highlight shell %}
jail -m name=apache allow.raw_sockets=1
{% endhighlight %}

install and start apache.  After this, on the host machine you should be able to curl 192.168.0.1 </i>
{% highlight shell %}
pkg ins apache24; sysrc apache24_enable="yes"; service apache24 onestart
{% endhighlight %}