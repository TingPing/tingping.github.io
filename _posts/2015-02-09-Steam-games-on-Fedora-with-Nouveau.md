---
layout: post
title: Steam games on Fedora with Nouveau
date: 2015-02-09 18:00:00
tags: [fedora howto linux]
---

Steam may not support Fedora nor Nouveau drivers, but surely it can work.


So I installed steam and tried to play a game and it failed to launch. When running it from the
terminal I get the output:

{% highlight sh %}
>LIBGL_DEBUG=verbose steam
...
libGL: pci id for fd 7: 10de:1200, driver nouveau
libGL: OpenDriver: trying /usr/lib/dri/tls/nouveau_dri.so
libGL: OpenDriver: trying /usr/lib/dri/nouveau_dri.so
libGL: dlopen /usr/lib/dri/nouveau_dri.so failed (~/.local/share/Steam/ubuntu12_32/steam-runtime/i386/usr/lib/i386-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.20' not found (required by /usr/lib/dri/nouveau_dri.so))
libGL error: unable to load driver: nouveau_dri.so
libGL error: driver pointer missing
libGL error: failed to load driver: nouveau
...
{% endhighlight %}

So for some reason nouveau is not liking the bundled glibc in the steam-runtime. Disabling the runtime requires tons of
system deps that aren't packaged or won't work with the games so I thought I'd just try using the system glibc. Note that
this could explode or be reverted in an update and I offer no warrenty of course.

{% highlight sh %}
cd ~/.local/share/Steam/ubuntu12_32/steam-runtime/i386/usr/lib/i386-linux-gnu/
mv libstdc++.so.6{,-backup}
ln -s /usr/lib/libstdc++.so.6 ./libstdc++.so.6

cd ~/.local/share/Steam/ubuntu12_32/steam-runtime/amd64/lib/x86_64-linux-gnu
mv libstdc++.so.6{,-backup}
ln -s /usr/lib64/libstdc++.so.6 ./libstdc++.so.6
{% endhighlight %}

And with that every game I had installed works fine.
