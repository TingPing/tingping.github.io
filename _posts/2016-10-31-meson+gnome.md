---
layout: post
title: Meson + Gnome Status Update
date: 2016-10-31 18:00:00
categories: linux
tags: gnome meson build linux builder gnome-builder build-system
---

This past month I have been doing a lot of work improving Meson for the Gnome ecosystem so as
a follow up to my [previous post](https://tingping.github.io/linux/2016/09/27/LAS-Hosted-by-GNOME.html) I thought I would
mention some of that progress and what still needs to be done.

<!--more-->

## Builder

### Building Builder

So most of my work has been focused around Gnome Builder this month. My first step was to get
Builder fully ported from Autotools to Meson as it is a fairly large and complex Gtk application
making use of most features anyone would need. This was a success (though not merged yet).

This took [19 PRs to Meson](https://github.com/mesonbuild/meson/pulls?utf8=%E2%9C%93&q=is%3Apr%20author%3ATingPing)
from me so far and more from other contributors. Some highlights:

- Add Yelp support
- Add ability to generate vapi files from gir files
- Add support for local its files with gettext

The single feature that was not ported to Meson was the building of c++ bindings (idemm).
The build files for mm seem extremely tied to autotools generating large m4 files and in
general is very complex so sadly I don't believe this will ever be ported to Meson
unless redesigned from the ground up.

Along the way I also ported Sysprof which after all of that work and me becoming familiar with Meson
was extremely easy and took a few hours in an evening. The Builder port was around ~3k LoC and the
Sysprof port was only 670 LoC for the build files.

### Builder Building

In order for Builder to truely move to Meson we need to replicate the integration it already has
with Autotools. Thankfully Meson already provides simple tools and files that provide easily
parsed json for the information we need such as compilation flags and build targets.
The [plugin](https://git.gnome.org/browse/gnome-builder/tree/plugins/meson) for this support
was fairly simple with nowhere near the complexitiy of the Autotools one of course and only
required [a single change](https://github.com/mesonbuild/meson/commit/6762d30c6a9b5da6a90f7c1eadc06d449dbd9a33)
upstream.

The integration we expose is also very basic at the moment and we will likely be able to expand
greatly on this in the future for features such as exposing specific targets, running tests, and configuration
which are all easy to get from Meson.

With that all in place we need to get new users to Meson so I also made a plugin to provide new project templates
for Meson which still needs some UI changes. Again this was far smaller and more concise than the Autotools
templates could ever be.

## Future in Gnome

I do think Meson is a good direction to go in and has the potential to improve correctness, simplicity, portability,
integration, and performance across the entire stack which is good for everybody involved. What we need to do now
is get this in the hands of developers so they can see first hand the differences.

The next project I would like to tackle is porting Gtk4 to Meson. *Most* of the developers are on board and this is
the time to try this migration. Baedert had already started this work and since I've done this multiple times now
I should be fairly efficient at porting at this point.

### Porting your project

I'd love to make a more in-depth article showing off what Meson can provide and how to port your project but until
then here are a few tips.

- Require *>= 0.36.0* which has a lot of fixes and can be installed with `pip`
- Diff install directories to notice any subtle differences (I suggest Meld)
- Check out existing projects like my Sysprof/Builder ports for real world examples
- For support ircs://chat.freenode.net/mesonbuild is very helpful; I'll be there
