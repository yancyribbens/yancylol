---
layout: post
title:  "If you're into evil you're a friend of mine"
date:   2022-05-05 18:09:00 +0300
categories: rust
---

Currently I'm running [evilwm](https://github.com/nikolas/evilwm) on top of [freebsd](https://www.freebsd.org) 13.0-RELEASE-p8.  After installing the port [evilwm](https://www.freshports.org/x11-wm/evilwm) I added the following to `~/.xinitrc`.

{% highlight nix %}
[ -f $HOME/.Xdefaults ] && xrdb $HOME/.Xdefaults
xsetroot -solid \#400040 -cursor_name top_left_arrow
/usr/local/bin/evilwm -snap 10
{% endhighlight %}

The most important piece is to launch `/usr/local/bin/evilwm` on startup.  `-snap 10` enables snap to window when you approach the boarder when resizing or moving a window.

I found the default xterm font to be very small and unreadable so I added to following to `~/.Xdefaults`

{% highlight nix %}
XTerm*background: black
XTerm*foreground: green
xterm*faceName: Monaco
xterm*faceSize: 13
{% endhighlight %}

Another side note is copy and paste work a bit different than other terminals I've used.  For example, just selecting text inside the terminal will save the contents to a clipboard, and then to paste into a different application, click the middle mouse button.  Another handy but related tip is that using evilwm, alt+left (and drag) click will move the window, however alt+middle click will resize the window.
