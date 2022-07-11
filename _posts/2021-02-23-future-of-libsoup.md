---
layout: post
title: Future of libsoup
tags: [gnome, igalia, webkit]
---

The libsoup library implements HTTP for the GNOME platform and is used by a wide range of projects
including any web browser using [WebKitGTK](https://webkitgtk.org). This past year we at [Igalia](https://igalia.com)
have been working on a new release to modernize the project and I'd like to share some details and get some
feedback from the community.

<!--more-->

## History

Before getting into what is changing I want to briefly give backstory to why these changes are happening.

The library has a long history, that I won't cover all of, being created in 2000 by Helix Code, which became
[Ximian](https://en.wikipedia.org/wiki/Ximian), where it was used in projects such as an email client (Evolution).

While it has been maintained to some degree for all of this time it hasn't had a lot of momementum
behind it. The library has maintained its ABI for 13 years at this point with ad-hoc feature additions
and fixes being often added on top. This has resulted in a library that has multiple APIs to accomplish
the same task, confusing APIs that don't follow any convention common within GNOME, and at times odd
default behaviors that couldn't be changed.

## What's Coming

We are finally breaking ABI and making a new libsoup 3.0 release. The goal is to make it a smaller,
more simple, and focused library.

Making the library smaller meant deleting a lot of duplicated and deprecated APIs, removing rarely used
features, leveraging additions to GLib in the past decades, and general code cleanup. As of today
the current codebase is roughly at 45,000 lines of C code compared to 57,000 lines in the last release
with over 20% of the project deleted.

Along with reducing the size of the library I wanted to improve the quality of the codebase. We now
have improved CI which deploys documentation that has 100% coverage, reports code coverage for tests,
tests against Clang's sanitizers, and the beginnings of automated code fuzzing.

Lastly there is ongoing work to finally add HTTP/2 support improving responsiveness for the whole
platform.

There will be follow up blog posts going more into the technical details of each of these.

## Release Schedule

The plan is to release libsoup 3.0 with the GNOME 41 release in September. However we will be releasing
a preview build with the GNOME 40 release for developers to start testing against and porting to. All feedback
would be welcomed.

Igalia plans on helping the platform move to 3.0 and will port GStreamer, GVFS, WebKitGTK and may help with some
applications so we can have a smooth transition.

For more details on WebKitGTK's release plans there is [a mailinglist thread over it](https://lists.webkit.org/pipermail/webkit-gtk/2021-February/003667.html).

The previous release libsoup 2.72 will continue to get bug fix releases for the forseable future but no new 2.7x
releases will happen.

## How to Help

You can build the current [head of libsoup](https://gitlab.gnome.org/GNOME/libsoup) to test now.

Installing it does not conflict with version 2.x however GObject-Introspection based applications may accidentally import the wrong version (Python for example needs `gi.require_version('Soup', '2.4')` and GJS needs `imports.gi.versions.Soup = "2.4";` for the old API) and you cannot load both versions in the same process.

A [migration guide](https://libsoup.org/ch02.html) has been written to cover some of the common questions as well as improved documentation and guides in general.

All bug reports and API discussions are welcomed on the [bug tracker](https://gitlab.gnome.org/GNOME/libsoup/-/issues).

You can also join the IRC channel for direct communication (be patient for timezones): [ircs://irc.gimp.net/libsoup](ircs://irc.gimp.net/libsoup).
