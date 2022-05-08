---
layout: post
title:  "Having things in jail"
date:   2022-05-05 18:09:00 +0300
categories: rust
---

To access filesystem constructs, for example a file or folder, requires a [vnode](https://man.openbsd.org/vnode.9) interface to be allocated and accessible in kernel memory.  However, in a unix jail, the contents of the host filesystem and vnode isnt't readily available.  The [nullfs](https://www.freebsd.org/cgi/man.cgi?mount_nullfs(8)) can be used to add a null layer which passes all vnode interface requests to the host filesystem.  Stackable filesystems allow a modular way to plug in different filesystems and use the stack to make the translation between different vnode interfaces.  More history is provided here: [UCLA Technical Report CSD-910056, Stackable Layers: an Architecture for File System Development.](https://www.freebsd.org/cgi/man.cgi?mount_nullfs(8)).

The null layer is the simplest layer and often used as the base skeleton to start from when creating a new layer interface.  Since in my case, both the jail and the host are the same filesystem type, the null layer can be used as follows to simply pass vnode operations between layers:c
{% highlight shell %}
sudo mount_nullfs ~/git/ /root/jails/tirana/home/tirana/git
{% endhighlight %}

The above command will graft the tree /root/jails/tirana/home/tirana/git on the host OS inside the jail at location ~/git inside of the jail.  That way, only one copy of the git repositories (and git keys) needs to be stored on the machine but can be accessed both inside the jail and outside on the host OS.  Here, the name of the jail is called tirana and exists in the directory /root/jails/ with the other jails on the host.

