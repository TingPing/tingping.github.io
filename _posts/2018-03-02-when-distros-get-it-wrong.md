---
layout: post
title: When distributions get it wrong
date: 2018-03-02 12:00:00
tags: [linux]
---

This is a brief rant about a poor packaging decision in Debian that was made recently that just
caught me by surprise and I felt like sharing.

<!--more-->

For context for those who do not know; I am the maintainer of [HexChat](https://hexchat.github.io)
which is a fork of the popular IRC client [XChat](http://xchat.org/). This fork is roughly 8 years old
in which time XChat has had 0 releases. As you can guess the fork exists
in its current form solely because XChat is dead. In those years the majority of distributions have
removed XChat from their repositories since it is an unmaintained project and HexChat is a logical
replacement.

So this story starts with Debian removing XChat from its repo on [2016-01-30](https://tracker.debian.org/news/744446)
which is not terrible in comparison to other distros but the problem arises when on [2017-08-08](https://tracker.debian.org/news/861826)
it was accepted back into the repository to my surprise. Since then the maintainer has [backported a few patches](http://metadata.ftp-master.debian.org/changelogs/main/x/xchat/xchat_2.8.8-13_changelog)
from HexChat including some CVE fixes and making UI changes to the input box totaling up to 44 patches as of today.
Since no other upstream exists this project is no longer XChat really it is a Debian specific fork
and due to timing this will land in Ubuntu 18.04 meaning this is theoretically "supported"
(by the community) until 2023.

I contacted the new maintainer just to get some sort of reasoning for this and his response honestly
confused me more:

>"I don't really have an answer for my adoption of xchat in Debian, it has been my first irc client, back in the days irc was really used, I loved it, I didn't love the hexchat necessary switch, but now I'm used to the new graphic, and I find it even superior. that said, I like to have a B plan in case hexchat stops working because of some new features requiring new systems, new libraries not available maybe on older pc (e.g.I maintain an Ubuntu ppa that builds xchat back to Ubuntu 14.04, I don't think hexchat can run on such older systems without patching, mainly due to the necessary switch to new libraries."

This package was added to Debian Unstable and Testing so any arguments about requiring libraries don't make sense
and surely he doesn't maintain a fork for every application he uses just as a fallback.

>"I was confused about the reintroduction, as well as you, but since the first upload, I got a lot of emails, thanking me bringing it back, and a backport request really minutes after it has hit unstable again. Other Debian Developers asked me to comaintain backports on older Debian  distributions, so I think I wasn't the only one feeling nostalgic of the old days, and old graphics :)."

It is indeed popular with the community as they love choices...

>"BTW, xchat is fixed for CVEs, and stable wrt libraries, I could even say that developing something increases the possibility to introduce new bugs :) Anyhow, unless you really find bugs / vulnerabilities in xchat/Debian, I would like to keep it in the archive for some more years, maybe until Ubuntu 14.04 goes End Of Life, or maybe until I find another good replacement for hexchat in case of breakages :)"

Now this reasoning is just completely wrong. I have touched nearly every corner of that codebase
and stared into its soul for many years and I probably know it better than anyone else (the
original author would need to refresh his memory). This codebase is terrible and it is full of
security problems and bugs. Yes there were only 2 new CVEs because I don't spend a third of my time
hacking on the project reporting new CVEs as more than a few of the thousands of bugs I have fixed are
potentially exploitable. The idea that only CVEs are real bugs is ridiculous and is not how software
development actually works.

I have no real conclusion for this story as I cannot solve it but I hope users of these distros
don't just accept that software in the repos is maintained or safe and I hope members of the Debian
and Ubuntu community can recognize that pulling in completely dead software into their repositories
is a bad idea.

EDIT: As a clarification I have nothing against forks of XChat or HexChat but they need to be done
responsibly as a clear new project with a dedicated developer and not a distro specific package
under the defunct projects name.