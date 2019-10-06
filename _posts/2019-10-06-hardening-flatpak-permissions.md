---
layout: post
title: Hardening Flatpak Permissions Over Time
tags: [flatpak]
---

One of the main goals of Flatpak is to sandbox applications but a common complaint is that
many packages add a lot of insecure permissions which is entirely valid. I'll be showing an example
of how over time many permissions now have secure alternatives.

<!--more-->

For this example I'll be using [Pithos](https://pithos.github.io) which is an application I maintain.  
It is an online radio player for Pandora.com.

These are the Flatpak permissions used for the application 2 years ago:

```json
  "finish-args": [
    "--share=ipc",
    "--share=network",
    "--socket=x11",
    "--socket=wayland",
    "--socket=pulseaudio",

    "--env=DCONF_USER_CONFIG_DIR=.config/dconf",
    "--filesystem=xdg-run/dconf",
    "--filesystem=~/.config/dconf:ro",
    "--talk-name=ca.desrt.dconf",

    "--talk-name=org.freedesktop.secrets",

    "--talk-name=org.freedesktop.Notifications",

    "--talk-name=org.gnome.SettingsDaemon",
    "--talk-name=org.mate.SettingsDaemon",

    "--talk-name=org.gnome.ScreenSaver",
    "--talk-name=org.cinnamon.ScreenSaver",
    "--talk-name=org.freedesktop.ScreenSaver",
    "--talk-name=com.canonical.Unity.Session",

    "--talk-name=org.kde.StatusNotifierWatcher",

    "--own-name=org.mpris.MediaPlayer2.pithos"
  ]
```

These are the possible\* permissions today:

```json
  "finish-args": [
    "--share=ipc",
    "--share=network",
    "--socket=fallback-x11",
    "--socket=wayland",
    "--socket=pulseaudio",

    "--talk-name=org.gnome.SettingsDaemon.MediaKeys",
    "--talk-name=org.mate.SettingsDaemon",

    "--talk-name=org.kde.StatusNotifierWatcher"
```

---

So lets go over each permission from the old manifest.

```json
    "--share=ipc", "--share=network", "--socket=x11", "--socket=wayland", "--socket=pulseaudio"
```

These are simply required for the application to function. The application will always require `network`
access and `wayland` access. We did replace `x11` with `fallback-x11` which means on Wayland X11 is blocked.
Eventually (hopefully within a year?) `pulseaudio` will have a similar solution for `pipewire` which will
be able to block things like being able to record audio.

```json
    "--filesystem=xdg-run/dconf", "--filesystem=~/.config/dconf:ro", "--talk-name=ca.desrt.dconf"
```

As of GLib 2.60.6+ this permission is simply removed and settings are stored locally. This prevents applications from
reading and writing settings for all software on the system.

```json
    "--talk-name=org.freedesktop.secrets"
```

In an upcoming `libsecret` release this will also store secrets locally preventing the application from being able to
read and write all stored secrets in the system keyring.

\* This is not in a stable release *yet*.

```json
    "--talk-name=org.freedesktop.Notifications"
```

A portal for notifications have been supported for a long time and the application simply uses the [GNotification](https://developer.gnome.org/gio/stable/GNotification.html) API
and just works.

```json
    "--talk-name=org.gnome.SettingsDaemon", "--talk-name=org.mate.SettingsDaemon"
```

These permissions are required for GNOME (and forks of GNOME like MATE) to bind media keys. Granting talk access to everything
the settings daemon can do is not ideal and lets the application mess with system wide settings. Years ago `gnome-settings-daemon`
improved this by splitting the features into their own namespaces so you only need to expose `org.gnome.SettingsDaemon.MediaKeys`
which should be relatively safe to talk to. Hopefully this goes away entirely eventually with a host side solution using MPRIS.

```json
    "--talk-name=org.gnome.ScreenSaver", "--talk-name=org.cinnamon.ScreenSaver", "--talk-name=org.freedesktop.ScreenSaver", "--talk-name=com.canonical.Unity.Session"
```

These were used to track the state of the screensaver to pause playback. This isn't the most dangerous thing in the world
but it does allow applications to activate the screensaver. GTK now supports the `GtkApplication::screensaver-active` property
which lets an application monitor the state without having extra permissions.

```json
    "--own-name=org.mpris.MediaPlayer2.pithos"
```

This was never a dangerous permission but as of Flatpak 1.0.4 you implicitly have permission to own `org.mpris.MediaPlayer2.$app_id`
and subnames so that is now used for our name.

```json
    "--talk-name=org.kde.StatusNotifierWatcher"
```

Lastly we do not have an alternative to this permission which still isn't ideal and could be used to spoof other applications
or exploit the host service which was likely never designed to handle untrusted data.

---

In conclusion there is certainly progress being made and this isn't the only application slowly tightening permissions. There
are of course still big holes where no alternative exists (most usage of `--filesystem`) but already today relatively secure
but still well integrated flatpak'd applications can and do exist.
