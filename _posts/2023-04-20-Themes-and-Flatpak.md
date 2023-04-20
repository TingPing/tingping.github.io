---
layout: post
title: How GTK3 themes work in Flatpak
tags: [gnome, flatpak]
---

There seems to be a lot of misinformation and low quality content out there on how to use a theme with Flatpak. So I'm going to break down how it all works.

## Clients

Before we talk about Flatpak we have to talk about how GTK3 itself decides what theme you use.

### Wayland

If you use Wayland this is very simple. It talks to `xdg-desktop-portal-gtk` to get your theme name from the host. The setting location on the host is in GSettings:

```
gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita-dark'
```

`gnome-tweaks` sets this value for you, so I'd recommend just using it. Other desktop tools *may* set this value in their respecting settings applications.

### X11

If you are on X11 it relies upon a standard known as [XSettings](https://www.freedesktop.org/wiki/Specifications/xsettings-spec/). How to configure this is less straightforward. On GNOME it uses `gsd-xsettings` as part of the `gnome-settings-daemon` project and it reads the GSettings value discussed above.

If you use a different daemon like `xsettingsd` you have to set `Net/ThemeName` in its configuration file. Other desktops may have their own daemon that need to be configured.

## Getting themes inside of Flatpak

Now that your theme is actually configured properly you need to get the theme files inside of flatpak. You often can run a single command, `flatpak update` and everything will work now. It reads the GSetting discussed above and downloads a packaged theme.

### Not packaged themes

If no package for the theme was found a common direction people go in is modifying permissions to add folders to the sandbox. I don't recommend this. Instead here is how you package your theme.

```
#!/bin/bash
DEFAULT_ARCH=`flatpak --default-arch`
THEME_NAME=`gsettings get org.gnome.desktop.interface gtk-theme | tr -d \'`
THEME_EXTENSION_DIR=~/.local/share/flatpak/extension/org.gtk.Gtk3theme.$THEME_NAME/$DEFAULT_ARCH/3.22

mkdir -p $THEME_EXTENSION_DIR

cp -r /usr/share/themes/$THEME_NAME/gtk-3.0/* $THEME_EXTENSION_DIR
```

This simply copies your theme into a private flatpak extension and everything should work fine as long as there aren't weird symlinks in your theme.
Replace `/usr/share/themes` with a different directory like `~/.themes` if needed.