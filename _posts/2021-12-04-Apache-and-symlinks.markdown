---
layout: post
title:  "Apache and symlinks"
date:   2021-12-04 14:02:00 +0100
categories: freebsd
---

On [11-17]({% post_url 2021-11-17-jailed-apache %}) I described setting up a Jail using FreeBSD and installing Apache.  While developing this blog, I can now spin up the Jail and test changes locally before deploying, and also use a Jail wherever this blog is hosted.  For the most part the process was smooth, and I copied over and setup my ed25519 keys and checked out the git repo.  Next, I setup a symlink:

Creates a symlink from my homepage Git repo to the directory served by Apache

{% highlight shell %}
ln -s ~/git/homepage/public-html /usr/local/www/apache24/data
{% endhighlight %}

However, after creating the symlink, I reload the webpage and receive a scary 403 Forbidden message.  Tailing the Apache error log confirms the problem "symbolic link not allowed or link target not accessible".

Show the end of the Apache error messages file.

{% highlight shell %}
tail /var/log/httpd-error.log
{% endhighlight %}

The error message made me think that the problem was the symlink, and after trying to change the permissions of the symlink target (since the symlink itself doesn't have file permissions), the problem persisted, no matter how permissive the permissions where for ~/git/homepage/public-html.  After a lot of head scratching, including trying different options in "/usr/local/etc/apache24/httpd.conf", I realized that it was the home directory ~ which in this case was /root where too restrictive.  The permissions must be permissive for all parent directories, and since this was setup in a Jail, I was using the default root account, relying on the Jail for security instead of user accounts. Once I created a limited user, and then created the symlink again, everything worked as expected.  As a side note, I believe the permissions issue was in part because even though Apache is running as root, and the directory is owned by root, when a new request arrives at port 80, Apache spawns a child process owned by www (not root) which causes the permissions error.
