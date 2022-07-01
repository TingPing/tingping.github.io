---
layout: post
title: WebExtension Support in Epiphany
tags: [gnome, igalia]
---

I'm excited to help bring WebExtensions to [Epiphany](https://wiki.gnome.org/action/show/Apps/Web) (GNOME Web) thanks to investment from my employer [Igalia](https://igalia.com). In this post, I'll go over a summary of how extensions work and give details on what Epiphany supports.

<!--more-->

Web browsers have supported extensions in some form for decades. They allow the creation of features that would otherwise be part of a browser but can be authored and experimented with more easily. They’ve helped develop and popularize ideas like ad blocking, password management, and reader modes. Sometimes, as in very popular cases like these, browsers themselves then begin trying to apply lessons upstream.

## Toward universal support

For most of this history, web extensions have used incompatible browser-specific APIs. This began to change in 2015 with Firefox adopting an API similar to Chrome's. In 2020, Safari also followed suit. We now have the foundations of an ecosystem-wide solution.

"The foundations of" is an important thing to understand: There are still plenty of existing extensions built with browser-specific APIs and this doesn't magically make them all portable. It does, however, provide a way towards making portable extensions. In some cases, existing extensions might just need some porting. In other cases, they may utilize features that aren't entirely universal yet (or, may never be).

## Bringing Extensions to Epiphany

With version 43.alpha Epiphany users can *begin* to take advantage of some of the same powerful and portable extensions described above. Note that there are quite a few APIs that power this and with this release we've covered a meaningful segment of them but not all (details below). Over time our API coverage and interoperability will continue to grow.

## What WebExtensions can do: Technical Details

At a high level, WebExtensions allow a private privileged web page to run in the browser. This is an invisible [Background Page](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/background) that has access to a [`browser`](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Browser_support_for_JavaScript_APIs) JavaScript API. This API, given permission, can interact with browser tabs, cookies, downloads, bookmarks, and more.

Along with the invisible background page, it gives a few options to show a UI to the user. One such method is a [Browser Action](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/browser_action) which is shown as a button in the browser's toolbar that can popup an HTML view for the user to interact with. Another is an [Options Page](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/options_ui) dedicated to configuring the extension.

Lastly, an extension can inject JavaScript directly into any website it has permissions to via [Content Scripts](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/content_scripts). These scripts are given full access to the DOM of any web page they run in. However content scripts don't have access to the majority of the `browser` API but, along with the above pages, it has the ability to send and receive custom JSON messages to all pages within an extension.

### Example usage

For a real-world example, I use Bitwarden as my password manager which I'll simplify how it roughly functions. Firstly there is a Background Page that does account management for your user. It has a Popup that the user can trigger to interface with your account, passwords, and options. Finally, it also injects Content Scripts into every website you open.

The Content Script can detect all input fields and then wait for a message to autofill information into them. The Popup can request the details of the active tab and, upon you selecting an account, send a message to the Content Script to fill this information. This flow does function in Epiphany now but there are still some issues to iron out for Bitwarden.

## Epiphany's current support

Epiphany 43.alpha supports the basic structure described above. We are currently modeling our behavior after Firefox's ManifestV2 API which includes compatibility with Chrome extensions where possible. Supporting ManifestV3 is planned alongside V2 in the future.

As of today, we support the majority of:

- `alarms` - Scheduling of events to trigger at specific dates or times.
- `cookies` - Management and querying of browser cookies.
- `downloads` - Ability to start and manage downloads.
- `menus` - Creation of context menu items.
- `notifications` - Ability to show desktop notifications.
- `storage` - Storage of extension private settings.
- `tabs` - Control and monitoring of browser tabs, including creating, closing, etc.
- `windows` - Control and monitoring of browser windows.

A notable missing API is `webRequest` which is commonly used by blocking extensions such as uBlock Origin or Privacy Badger. I would like to implement this API at some point however it requires WebKitGTK improvements.

For specific API details please see [Epiphany's documentation](https://gitlab.gnome.org/GNOME/epiphany/-/blob/master/src/webextension/README.md).

What this means today is that users of Epiphany can write powerful extensions using a well-documented and commonly used format and API. What this does not mean is that most extensions for other browsers will just work out of the box, at least not yet. Cross-browser extensions are possible but they will have to only require the subset of APIs and behaviors Epiphany currently supports.

## How to install extensions

This support is still considered experimental so do understand this may lead to crashes or other unwanted behavior. Also please report issues you find to Epiphany rather than to extensions.

You can install the development release and test it like so:

```
flatpak remote-add --if-not-exists gnome-nightly https://nightly.gnome.org/gnome-nightly.flatpakrepo
flatpak install gnome-nightly org.gnome.Epiphany.Devel
flatpak run --command=gsettings org.gnome.Epiphany.Devel set org.gnome.Epiphany.web:/org/gnome/epiphany/web/ enable-webextensions true

# Due to a temporary bug you need to run this:
mkdir -p ~/.var/app/org.gnome.Epiphany.Devel/config/epiphany/web_extensions/
```

You will now see **Extensions** in Epiphany's menu and if you run it from the terminal it will print out any message logged by extensions for debugging. You can download extensions most easily from [Mozilla's website](https://addons.mozilla.org/en-US/firefox/extensions/).
