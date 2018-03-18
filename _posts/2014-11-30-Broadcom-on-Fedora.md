---
layout: post
title: Broadcom on Fedora
tags: [fedora howto linux]
---
A few months ago I purchased a [Broadcom BCM4322 wifi card](http://www.amazon.com/gp/product/B00AEEL1BY/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1)
for my laptop to use in OSX. I had heard issues about Broadcom on Linux but I knew drivers existed
so how hard could it be?

<!--more-->

Upon booting Fedora 21 with the new card it did not work as expected. So I immediately searched through
RPMFusion for a broadcom driver, found [broadcom-wl](http://download1.rpmfusion.org/nonfree/fedora/releases/21/Everything/x86_64/os/repoview/broadcom-wl.html), installed it, and everything worked. Mission success right? Sadly not.

Over the next month or so the experience of this driver was absolutely terrible. It would disconnect, have poor performance, take forever to connect, be slow waking from sleep, etc. So I finally decided to find out why
this was.

It turns out there are two drivers for the card I have; The open source one, b43, and the proprietary one, broadcom-wl. The proprietary one is the one everybody refers to as terrible and should only be used as a last
resort but I assume many mistakingly use that first. You can check if your chip is supported by b43 here: [http://linuxwireless.sipsolutions.net/en/users/Drivers/b43/](http://linuxwireless.sipsolutions.net/en/users/Drivers/b43/).

So why didn't b43 load automatically? It requires installing the firmware for your card. For some reason this
is not packaged in RPMFusion, maybe I will solve that someday, so it had to be built manually. Lucky for
anybody reading this I have done the hard work for you. You can install the [b43-firmware](https://dl.tingping.se/fedora/repoview/b43-firmware.html) package from my repo.
