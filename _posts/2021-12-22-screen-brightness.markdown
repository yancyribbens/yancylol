---
layout: post
title:  "Screen Brightness"
date:   2021-12-22 14:02:00 +0100
categories: freebsd
---

FreeBSD is a great operating system imo, although I'm still fine tuning my setup and sharpening knives.  I notice every time I turn on my computer to work I spend at least 15-20 seconds increasing the brightness before working no matter what.  The steps I use to increase the brightness:

loading kernel modules requires privilege escalation (and typing a password)

{% highlight shell %}
su
{% endhighlight %}

load the acpi_video driver into the kernel using kldload

{% highlight shell %}
kldload acpi_video
{% endhighlight %}

Loading the driver every time was cumbersome, and while it seemed like it should be straight forward to load the module at boot, adjusting the brightness took a bit of searching and experimentation.  First, I added the following to `/boot/loader.conf` which will provide the driver during the boot process.

{% highlight shell %}
acpi_video_load="YES"
{% endhighlight %}

Lastly, there are two knobs for setting the brightness depending on if the computer is plugged in or running on bats.  For myself I just set these both to 100 since that seems to be my default (volume 11/10).  To adjust these settings, I add the following to `/etc/sysctl.conf`:

{% highlight shell %}
hw.acpi.video.lcd0.economy=100
hw.acpi.video.lcd0.fullpower=100
{% endhighlight %}

While experimenting with different boot configurations I timed the time it takes to boot on this Thinkpad T480s, and currently it clocks in at around 1 minute.  There are some things I could remove from the boot process I think like sendmail which I don't use but that's a post for a different day.
