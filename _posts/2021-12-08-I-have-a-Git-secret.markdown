---
layout: post
title:  "I have a Git secret"
date:   2021-12-08 14:02:00 +0100
categories: freebsd 
---

Git-secret is a little-known tool with neat properties.  It's a great example of how someone can maintain sovereignty over their personal data while still using the existing cloud infrastructure.  For example, as I travel, I keep a journal of activities and other things going on in life.  While I maintain this journal behind a private Git repo, I toyed with the idea of making my journal public, but felt too shy to share all of my thoughts with the world, even if there was a way to keep my true identity private.  Why then should I store my journal on Github allowing the Github folks access to my private journal.  There's a few ways I could keep my journal private using Git, and one possibility is to host a private Git server and administer it or use git-secret.

Other data I store on Git that I want to keep private is all the SSIDs and passwords.  If someone wanted to know about my whereabouts, this file of SSID (for example local coffee shop hotspots) could be used to draw a pattern of locations I frequent.  I can minimize the surface area of what's revealed to the cloud service providers by using git-secret.

Git-secret is provided separately from git.

Installs git-secret using <a href="https://www.freebsd.org/cgi/man.cgi?query=pkg-install">pkg-install</a>

{% highlight shell %}
pkg ins git-secret
{% endhighlight %}

Git secret works in concert with gpg, and while I make no guarantee on the crypto behind gpg because I haven't reviewed it personally, the high level workflow works as follows.

Tell which gpg key to associate with

{% highlight shell %}
git secret tell your@gpg.email
{% endhighlight %}

Creates a directory called .gitsecret and also adds .gitsecret/keys/random_seed to .gitignore so that you can't accidentally share your private seed.

{% highlight shell %}
git secret init
{% endhighlight %}

Add all files to .gitignore that you want to keep hidden

{% highlight shell %}
vim .gitignore
{% endhighlight %}

Begin tracking a file to be hidden.  Note that this file can not have been added to the repo previously.

{% highlight shell %}
git secret add file
{% endhighlight %}

Hide the file named file.

{% highlight shell %}
git secret hide
{% endhighlight %}

At this point we can use the familiar git workflow.  The example above will create a file ending in .secret which can be safely checked in and pushed to the cloud.  Only the people that have keys associated with this repo are allowed to view and update the contents.  In the above example, there is only one person, but in theory there could be many people with keys to this repo and the cloud provider has no way to know what data is stored.  Whenever changes to this file are made, the command "git secret hide" will encrypt the file again and can be committed to replace the current encrypted version.
