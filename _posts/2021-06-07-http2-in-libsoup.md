---
layout: post
title: HTTP/2 in libsoup3, WebKitGTK, and Epiphany
tags: [gnome, igalia, webkit]
---

The latest development release of libsoup 3, 2.99.8, now enables HTTP/2 by default.
So lets look into what that means and how you can try it out.

<!--more-->

## Performance

In simple terms what HTTP/2 provides for improved performance is more efficient network
usage when requesting multiple files from a single host. It does this by avoiding making
new connections whenever possible and over that single connection allowing multiple
requests to happen at the same time.

It is easy to imagine many workloads this would improve, such as `flatpak` downloading
a lot of objects from a single server.

Here are some examples in Epiphany:

#### gophertiles

This is a benchmark made to directly test the best case for HTTP/2. As you can see
in the inspector (which has been improved to show more network information)
you can see HTTP/2 creates a single connection and completes in 229ms. HTTP/1 on the
other-hand creates 6 connections taking 1.5 seconds. This all happens on a network which
is a best case for HTTP/1, a low latency wired gigabit connection; As network latency
increases HTTP/2's lead grows dramatically.

![browser screenshot using http2](/images/gophertiles-http2.png)

![browser screenshot using http1](/images/gophertiles-http1.png)

#### Youtube

For a more real world example Youtube is a great demo. It hosts a lot of files for a webpage
but it isn't a perfect benchmark as it still involves multiple hosts that don't share
connections. HTTP/2 still has the slight lead, again versus HTTP/1's best case.

![inspector screenshot using http2](/images/youtube-http2.png)

![inspector screenshot using http1](/images/youtube-http1.png)

## Testing

This work is all new and we would really like some testing and feedback. The easiest
way to run this yourself is with this custom [Epiphany Flatpak](https://dl.tingping.se/flatpak/epiphany-canary.flatpak) (sorry for the slow download server, and it will not
work with NVidia drivers).

You can get useful debug output both through the WebKit inspector (ctrl+shift+i) and
by running with `--env='G_MESSAGES_DEBUG=libsoup-http2;nghttp2'`.

Please report any bugs you find to [https://gitlab.gnome.org/gnome/libsoup/issues](https://gitlab.gnome.org/gnome/libsoup/issues).
