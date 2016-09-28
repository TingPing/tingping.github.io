---
layout: post
title: Libre Application Summit hosted by GNOME
date: 2016-09-27 18:00:00
categories: linux
tags: gnome event conference
---

The first [Libre Application Summit](http://las.gnome.org/) in Portland just finished this week and I had the chance to go.

<!--more-->

This was the first ever conference I've been to and it was great to meet everybody I knew from online in person
such as Christian Hergert, Garrett Regier, Sriram Ramkrishna, and so many more who were all very welcoming.
Portland was also a great setting for the conference with lots of beautiful sites and I hope to be able to visit
again. With that out of the way onto more technical topics.

## Flatpak

So going into this I already had a head start on [Flatpak](http://flatpak.org/); I have [already packaged](https://github.com/TingPing/flatpak-packages)
most projects I maintain and already [host a repo](https://dl.tingping.se/flatpak/) for them. It was great to see how
everybody else was equally sold on Flatpak though especially Andrew Walton's talk about VMware's current distribution
nightmare and how they see Flatpak as the clear solution to that.

## Meson

For those of you who do not know [Meson](https://github.com/mesonbuild/meson) is a new build system which has made some noise
in the GNOME community over the past few years. I have a more positive opinion of Autotools than most so I was a bit skeptical
that Meson could replace it entirely. Jussi Pakkanen gave two talks over Meson and speaking with him over the week I am more
sold on the idea of migrating to it.

### Windows

So one major pain in building is Windows; Currently the [HexChat Project](https://hexchat.github.io) maintains a [custom build
script](https://github.com/hexchat/gtk-win32) written in PowerShell which is as bad as it sounds. Meson has the concept of [Wraps](https://github.com/mesonbuild/meson/wiki/Wrap-dependency-system-manual)
which allow easily building an entire stack from source which is valuable on Windows. I have yet to use it in practice but it
is certainly on my todo list.

### Builder Integration

I have occasionally contributed to [GNOME Builder](https://wiki.gnome.org/Apps/Builder) and as an IDE a difficult task is to
sanely parse and modify an Autotools projects. Meson unlike Autotools exposes JSON data for compilation flags and targets which
can allow tools to get actually usable information. I have already started work integrating this [in a branch](https://git.gnome.org/browse/gnome-builder/log/?h=wip/tingping/meson)
that is a work in progress but already it can build and run projects. The internal API of Builder is a bit Autotools
focused in this area, there are some Python binding problems, and we need to work with Meson to expose more information
but it should land this cycle.

## IRC

I did briefly discuss IRC at a [BoF](https://en.wikipedia.org/wiki/Birds_of_a_feather_(computing)) mentioning the current state
of initiatives like [IRCv3](http://ircv3.net). The end conclusion is just that we need to get in contact with the infrastructure
of GNOME to push for more modern server features and possibly expose things like chat history which is already doable at the
protocol level.

## Monetization

There was a BoF over how to monitize applications and I was honestly disappointed by this discussion. Technologies like Flatpak
and the future website of FlatHub, a hosted service for building and distributing flatpaks, help this but I don't feel like any new
useful strategies were brought up and the issue of getting the community at large to *want* to donate/pay, especially for existing
projects, is a bigger blocker than some realize. This is just something that will need to be discussed further.

<div style="text-align:center" markdown="1">
![GNOME Foundation Sponsored Badge](https://wiki.gnome.org/Travel/Policy?action=AttachFile&do=get&target=sponsored-badge-simple.png)
</div>
