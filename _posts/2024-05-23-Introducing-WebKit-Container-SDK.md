---
layout: post
title: Introducing the WebKit Container SDK
tags: [igalia, webkit]
---

Developing [WebKitGTK](https://webkitgtk.org) and [WPE](https://wpewebkit.org/) has always had challenges such as the amount of dependencies or it's fairly complex C++ codebase which not all compiler versions handle well. To help with this we've made a new SDK to make it easier.

<!--more-->

### Current Solutions

There have always been multiple ways to build WebKit and its dependencies on your host however this was never a great developer experience. Only very specific hosts could be "supported", you often had to build a large number of dependencies, and the end result wasn't very reproducable for others.

The current solution used by default is a Flatpak based one. This was a big improvement for ease of use and excellent for reproducablity but it introduced many challenges doing development work. As it has a strict sandbox and provides read-only runtimes it was difficult to use complex tooling/IDEs or develop third party libraries in it.

The new SDK tries to take a middle ground between those two alternatives, isolating itself from the host to be somewhat reproducable, yet being a mutable environment to be flexible enough for a wide range of tools and workflows.

### The WebKit Container SDK

At the core it is an Ubuntu OCI image with all of the dependencies and tooling needed to work on WebKit. On top of this we added some scripts to run/manage these containers with podman and aid in developing inside of the container. It's intention is to be as simple as possible and not change traditional development workflows. 

You can find the SDK and follow the quickstart guide on our GitHub: [https://github.com/Igalia/webkit-container-sdk](https://github.com/Igalia/webkit-container-sdk)

The main requirements is that this only works on Linux with podman 4.0+ installed. For example Ubuntu 23.10+.

In the most simple case, once you clone `https://github.com/Igalia/webkit-container-sdk.git`, using the SDK can be a few commands:

```sh
source /your/path/to/webkit-container-sdk/register-sdk-on-host.sh
wkdev-create --create-home
wkdev-enter
```

From there you can use WebKit's build scripts (`./Tools/Scripts/build-webkit --gtk`) or CMake. As mentioned before it is an Ubuntu installation so you can easily install your favorite tools directly like VSCode. We even provide a `wkdev-setup-vscode` script to automate that.

### Advanced Usage

#### Disposibility

A workflow that some developers may not be familiar with is making use of entirely disposable development environments. Since these are isolated containers you can easily make two. This allows you to do work in parallel that would interfere with eachother while not worrying about it as well as being able to get back to a known good state easily:

```
wkdev-create --name=playground1
wkdev-create --name=playground2

podman rm playground1 # You would stop first if running.
wkdev-enter --name=playground2
```

#### Working on Dependencies

An important part of WebKit development is working on the dependencies of WebKit rather than itself, either for debugging or for new features. This can be difficult or error-prone with previous solutions. In order to make this easier we use a project called [JHBuild](https://gnome.pages.gitlab.gnome.org/jhbuild/) which isn't new but works well with containers and is a simple solution to work on our core dependencies.

Here is an example workflow working on GLib:

```
wkdev-create --name=glib
wkdev-enter --name=glib

# This will clone glib main, build, and install it for us. 
jhbuild build glib

# At this point you could simply test if a bug was fixed in a different versin of glib.
# We can also modify and debug glib directly. All of the projects are cloned into ~/checkout.
cd ~/checkout/glib

# Modify the source however you wish then install your new version.
jhbuild make
```

Remember that containers are isoated from each other so you can even have two terminals open with different builds of glib. This can also be used to test projects like Epiphany against your build of WebKit if you install it into the JHBUILD_PREFIX.

### To Be Continued

In the next blog post I'll document how to use VSCode inside of the SDK for debugging and development.