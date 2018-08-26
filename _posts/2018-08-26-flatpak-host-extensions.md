---
layout: post
title: Using host Nvidia driver with Flatpak
tags: [flatpak]
---

This post is going to be a bit of a deep dive into how GL driver extensions work on
Flatpak, why they work the way they do, and how we can best use them moving forward.
Some of this information is useful for distro packagers and of course just anybody
interested in Flatpak details.

<!-- more -->

Just as a quick recap Flatpak *does not use host libraries*. This is required to be
portable and reproducable on nearly any Linux distro. This even applies to very
important libraries like libgl but this is rarely a problem as Mesa supports a wide
range of hardware. The exception to this is properietary drivers which for desktop
users means Nvidia. This isn't necessarily a problem since we can just package more
drivers but Nvidia's userspace drivers must be the same version as the kernel driver
which becomes a bit of a packaging nightmare because then we have to package every
version that is in use.

The logical solution to this is for the Flatpak GL driver to come from the same place
that your host GL driver did. Flatpak supports "unmanaged extensions" allowing loading
extensions installed externally into `/var/lib/flatpak/extension` and
`$XDG_DATA_HOME/flatpak/extension`. This does make my previous paragraph a lie but
it places trust in the host to be responsable and understand how ABI works to keep
things portable which we will get into later.

Before we get into packaging the Nvidia driver on the host for this lets look at how
the GL driver extension point specifically works. This extension point is [defined](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/blob/18.08/elements/flatpak-images/platform.bst#L21-31)
as such:

```yaml
'Extension org.freedesktop.Platform.GL':
    versions: "18.08;1.4"
    version: "1.4"
    directory: "%{lib}/GL"
    subdirectories: "true"
    no-autodownload: "true"
    autodelete: "false"
    add-ld-path: "lib"
    merge-dirs: "vulkan/icd.d;glvnd/egl_vendor.d"
    download-if: "active-gl-driver"
    enable-if: "active-gl-driver"
```

I will point out the important parts of this later but I will just cover what
`active-gl-driver` means really quick. The `-if` properties run custom checks within
`flatpak` to know if an extension should be downloaded or enabled. This check specifically
looks up your nvidia version from the kernel and maps that to a string e.g. `nvidia-396-54`.
What your current version is can be checked with `flatpak --gl-drivers` where you will notice
`host` and `default` are always inserted. This value can also be overriden with the env var 
`FLATPAK_GL_DRIVERS`.

So here is a real world example of me packaging this on Fedora: [negativo17/nvidia-driver](https://github.com/negativo17/nvidia-driver/pull/57).
The path it ends up installed into is: `…/org.freedesktop.Platform.GL.host/x86_64/1.4`.
As you can see `host` is used here because it is always in the default driver list and thus will always get used first.
The `1.4` version might also be confusing since the `1.4` runtime is ancient and nobody uses it. The reason this is used
is because the Nvidia driver has no ABI requirements outside of libc basically and will run on every freedesktop runtime
that exists so far. That guarentee does not exist for all drivers and must be used cautiously. The layout of files
ends up being this because of `merge-dirs` and `add-ld-path` shown above:
```
├── glvnd
│   └── egl_vendor.d
│       └── 10_nvidia.json
├── lib
│   ├── libGLX_nvidia.so.396.54
│   └── All libs omitted for brevity…
├── OpenCL
│   └── vendors
│       └── nvidia.icd
└── vulkan
    └── icd.d
        └── nvidia_icd.json
```

That should cover all the details for host extensions but this extension point is
also still very useful for normal flatpaked extensions. There is a [MR for mesa-git](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/merge_requests/458)
that would end up being used by setting `FLATPAK_GL_DRIVERS=mesa-git` to try out
newer versions.

If you are a distro packager I do believe it makes sense to start making a nvidia
package for the Flatpak extension especially for beta versions. This should probably
not be used to package host versions of Mesa though because they link to host
libraries outside of libc. There is some work done to make this possible but most
distro packages are not acceptable for this usage and there is rarely an upside as
fdo 18.08 might even have newer Mesa versions than some distros and can get updates.
