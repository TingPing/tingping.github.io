---
layout: post
title: How to design a status icon library in 2019
tags: [linux]
---

So the year is 2019 and people still use system trays on their desktops. I'm not going to get into
if that is good or bad but lets discuss an implementation to sanely use them and what existing ones
get wrong. I hope this is somewhat informative to developers using or implementing status icons.

<!--more-->

Lets start with some high level goals:

- Don't use X11

   I'll start with this because its simple and everybody knows it by now. If you want to set a tray you can't
   use X11 because not everybody uses X11. So you need a different solution and on Linux that is a DBus service
   that applications talk to.

- Safely handle and expose if the tray works

   There are always going to situations where the tray will not work such as using a desktop that doesn't support
   it or bugs like the service crashing and going away. This should always be reported to the application as soon
   as possible as this avoids issues like an application having a hide window feature but no tray to show it.

- Be designed for a sandboxed future

   We are now at a point where desktop software is getting sandboxed so that means certain actions are limited such
   as reading/writing files on the host, talking to or owning arbitrary DBus names, having a PID that means anything
   to the host, etc. On top of that applications should expect a permission system where they are denied using a tray.

- Recommended features

   This is more opinionated but I'd say you need to support:

   - Showing an icon and changing it (with scaling ideally)
   - Reacting to clicks and maybe scrolling
   - Exporting a menu to be shown


Ok I think that covers the basics lets look at the real world solutions:

### - GtkStatusIcon (GTK)

  - ❌ Uses X11
  - ✔️ It does expose if the tray is "embedded" or not (almost nobody listens to this).
  - ❌ Sandbox friendly (its X11)
  - ✔️ Features

  Nobody should use this API in new code if you want to target non-X11 but this is the classic API
  everybody has lived with for years and it got the job done. Since it no longer exists in GTK4
  due to not being portable a replacement had to come.

### - AppIndicator (Canonical)

  - ✔️ Uses DBus (but falls back to X11)
  - ❌ Failure not exposed by default
  - ❓ It is kinda sandbox friendly
  - ✔️ Features

  While nobody should be using the DBus service anymore many use the libappindciator library which behind the scenes uses StatusNotifier these days. It has a complete
  feature-set for the most part though menu handling is overly complex.

  There are quite a few problems with this implementation however. It is designed to silently fail by default; It falls back to X11 automatically and unless you override
  the fallback handling XEmbed failing yourself (maybe literally nobody does) you will not know it did. It does tell if you if the DBus portion is connected but as long
  as it fallsback you don't know what is going on.

  It isn't quite sandbox friendly because it does require talking to an insecure service and it allows referencing icons by path. However it doesn't require bus
  name ownership which is a positive.

  Lastly the library is entirely unmaintained for many years and libdbusmenu is just disgusting, so its not something I'd recommend in new code.

### - StatusNotifier (KDE)

  - ✔️ Uses DBus
  - ✔️ You can detect failure if you try
  - ❌ Not sandbox friendly
  - ✔️ Features

  This is the de-facto standard at the moment that many people are using now. It has its positives and its negatives.

  The biggest problem with this API is that it is clearly not designed with a sandbox in mind. Quoting the spec:

  > Each application can register an arbitrary number of Status Notifier Items by registering on the session bus the service org.kde.StatusNotifierItem-PID-ID,
  > where PID is the process id of the application and ID is an arbitrary numeric unique identifier between different instances registered by the same application.

  So this bad on a few levels. For one it requires name ownership in the KDE namespace which I personally do not understand why. Flatpak blocks all ownership except for your app-id.
  I'm not even sure a well-known name is required at all.

  Secondly it requires putting your PID in the name which is a no-go for any PID namespace on Linux as they will almost always have the same values inside the sandbox.

  Lastly it has the same issues AppIndicator exposed such as talking to an insecure service, referencing icons by path, etc. It also reuses the DBusMenu spec for exporting
  menus which is still completely awful and unmaintained.

  I'd say it is the best solution we have but it needs improving.

### - StatusIcon (Linux Mint)

  - ✔️ Uses DBus (but falls back to X11)
  - ❌ Failure not exposed
  - ❌ Not sandbox friendly
  - ❌ Features

  So last comes Mint with a new interface which repeats the issues of the past.

  Failure is not exposed at all so applications have no idea if it works.

  It has all of the sandbox problems of StatusNotifier nearly copy paste. It requires name ownership, it uses your PID in said ownership, etc.

  In the end it does not expose as many features as the previous solutions as you can only set an icon, label, and tooltip. I understand why to a degree
  for menus because libdbusmenu is seriously evil but they could have went a more minimal approach and reused GMenu in GLib which perfectly
  handles exporting a menu over DBus.

  This announcement is clearly why I am writing this as I hope they will consider some of the sandboxing cases and better expose if a tray
  is working in the future. I just do not want to see past mistakes repeated and for adoption to happen before they are solved.

  P.S. Don't use the `org.x` namespace and don't do blocking IO.

### - What I want to see (or maybe create)

  - Supports features mentioned above
  - Backed by DBus service
    - This must be a robust service, validating the sent icon data in a sandbox
      - All icons must be sent as data or icon-names, no paths allowed
    - Not require well-known name ownership outside of app-id namespace
      - Uniqueness is up to the application already
    - Support more minimal menus than DBusMenu (via GMenu IMO, but anything similar)
    - Be exposed by `xdg-desktop-portal` which handles permissions
  - Library that exposes if the tray is working or not
    - Also don't fallback to XEmbed, at least not silently
