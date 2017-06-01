---
layout: post
title: Flatpak now supports themes
date: 2017-05-11 12:00:00
tags: [linux flatpak gtk themes]
---

A common complaint against Flatpak users have is that it does not have a native
look because it doesn't match your hosts Gtk theme. As of now this will hopefully
become an issue of the past.

<!--more-->

### Background

For those not familiar with Flatpak a brief summary on why themes didn't work
in the first place is that all flatpaks are sandboxed and do not have the ability
to read data files or libraries in `/usr`. This is done so it is portable across
a wide range of distributions in a reproducable environment but has the downside
of not being able to pick up themes.

The solution to this was simply to package themes as relying upon the
host to have the correct version for every flatpak defeats the portability
benefits it provides. These themes are provided as [extensions](https://github.com/flatpak/flatpak/wiki/Extensions)
to the Gnome runtime.

### Installing themes

The current themes are packaged in the [flathub](https://flathub.org) repository which you can add by running:
```
flatpak remote-add flathub https://flathub.org/repo/flathub.flatpakrepo
```

To see a list of currently packaged themes you can run:
```
flatpak remote-ls flathub | grep org.gtk.Gtk3theme
```

The list as of posting is:

- org.gtk.Gtk3theme.Ambiance
- org.gtk.Gtk3theme.Arc (and [-solid variants](https://github.com/horst3180/arc-theme/pull/779))
- org.gtk.Gtk3theme.Arc-Dark
- org.gtk.Gtk3theme.Arc-Darker
- org.gtk.Gtk3theme.Breeze
- org.gtk.Gtk3theme.Breeze-Dark
- org.gtk.Gtk3theme.Greybird

You can install them with `flatpak install flathub org.gtk.Gtk3theme.Foo`.

These themes should cover most KDE, Unity, Budgie, and some XFCE users. If there are any
other popular themes you would like to request feel free to tell me on IRC or Reddit. If
you are a theme author and would like to maintain it yourself stop by [#flatpak on freenode](ircs://chat.freenode.net/flatpak)
and we can help you make one.

#### Notes

This requires Flatpak 0.8.4+ and applications using up to date `org.gnome.Platform` 3.24+.

The same solution exists for icon themes and more will be packaged soon.
They can be searched with `flatpak remote-ls flathub | grep org.freedesktop.Platform.Icontheme`

This process will likely be automated by Gnome-Software at some point. The `flatpak` tool
doesn't depend upon Gtk so it will not automatically detect your theme.

Themes are picked up via GtkSettings, which means you require a settings-daemon running to
set this. On GNOME this will already be running but for other desktops you can run
`gsd-xsettings` directly in your libexec directory provided by `gnome-settings-daemon`.

If you use the *Global Dark Theme* option in `gnome-tweak-tool` it will not work as that
simply writes to `settings.ini` which isn't available in the sandbox. Use dark versions
of the theme instead if they exist.
