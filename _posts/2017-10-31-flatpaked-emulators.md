---
layout: post
title: Status of emulators Flatpak'd
tags: [flatpak]
---

Console emulators have always been a great use-case for Flatpak since they are a
pain to build, sometimes proprietary, and not allowed in some distributions. As
such I, with some help, have been packaging a few of the more popular and maintained emulators
lately and thought I would mention them and keep a semi-maintained table for
future reference.

<!--more-->

All applications in my repository are considered experimental, are likely built
directly from git, and may or may not work or be supported.

Forgive the current version of this websites theme not rendering the table
very well, that is fixed in the next version.

| Console | Project | Application ID | Repository |
|---------|---------|----------------|------------|
| Nintendo | [Nestopia-UE](http://0ldsk00l.ca/nestopia/) | ca._0ldsk00l.Nestopia | [flathub](https://flathub.org) |
| Super Nintendo | [Snes9x](http://www.snes9x.com/) | com.snes9x.Snes9x | [flathub](https://flathub.org) |
| Game Boy Advance | [mGBA](https://mgba.io/) | io.mgba.mGBA | [flathub](https://flathub.org) |
| GameCube/Wii | [Dolphin](https://dolphin-emu.org/) | org.DolphinEmu.dolphin-emu | [flathub](https://flathub.org) |
| 3DS | [Citra](https://citra-emu.org/) | org.citra_emu.Citra | [tingping](https://dl.tingping.se/flatpak/tingping.flatpakrepo) |
| Wii-U | [Cemu](http://cemu.info/) | info.cemu.Cemu | [tingping](https://dl.tingping.se/flatpak/tingping.flatpakrepo) |
| PlayStation 2 | [PCSX2](https://pcsx2.net/) | net.pcsx2.PCSX2 | [flathub](https://flathub.org) |
| PlayStation Portable | [PPSSPP](http://ppsspp.org/) | org.ppsspp.PPSSPP | [flathub](https://flathub.org) |
| PlayStation 3 | [RPCS3](https://rpcs3.net/) | net.rpcs3.RPCS3 | [tingping](https://dl.tingping.se/flatpak/tingping.flatpakrepo) |
| Muti-Console\* | [RetroArch](https://www.libretro.com/) | org.libretro.RetroArch | [flathub](https://flathub.org) |


<small>\* RetroArch supports over a dozen consoles but some highlights not handled elsewhere: DS, Nintendo 64, PlayStation, Dreamcast, Game Boy, Game Boy Color</small>

This list would be longer but as mentioned emulators are often quite a pain to build or distribute.
Some failures have been:

- [epsxe](http://www.epsxe.com/) for the PS1 which is a proprietary core and doesn't
  work with some third-party awful plugins (some foss, all bad) on top.
- [PCSX-Reloaded](https://pcsxr.codeplex.com/) which works but is a dead project.
- [Mupen64Plus](http://mupen64plus.org/) which has no UI, third-party ones exist
  which I might investigate.
- [Play!](http://purei.org/) for the PS2 which is still early in development.
- [Higan](https://byuu.org/) required some patches but works fine I was just petty
  and too annoyed by its UI to package it.
- [DeSmuME](https://desmume.org/) for the DS had some C++ build errors I didn't feel
  like getting into as [melonDS](http://melonds.kuribo64.net/) seems like a more
  promising future there anyway.

So that is all so far; Let me know if there are some emulators you'd like packaged.
