---
layout: post
title: How to force AMDGPU to use full RGB range
tags: [linux]
---

I recently got an AMD GPU but sadly unlike i915 (Intel) and nvidia it doesn't expose any way of setting the full RGB range which my monitor wants instead of limited. There are a dozen issues from the kernel, xorg, wayland compositors, etc to get this exposed somehow but
it still isn't today.

Thankfully there is a hacky solution of using a custom [EDID](https://en.wikipedia.org/wiki/Extended_Display_Identification_Data) and this solution works on Wayland as well as Xorg (it should work with other drivers also). I was not actually the one to figure this out so shout out to [parkerlreed](https://www.reddit.com/r/AMDHelp/comments/5e966r/ycbcr444_pixel_format_amdgpupro/) on reddit. This post is just to make it more visible and I packaged the tool used to make this easier.

1. Find the `edid` file used by the connector: `find /sys/devices/pci*/ -name edid`
    - For convenience copy to home: `cp /sys/devices/.../edid ~/edid.bin`
2. Install wxEDID: `flatpak install https://dl.tingping.se/flatpak/wxedid.flatpakref`
3. Open the found file in wxEDID and edit it to disable YCbCr support:
    - SPF: Supported features -> change value of `vsig_format` to `0b00`
    - CHD: CEA-861 header -> change the value of `YCbCr420` and `YCbCr444` to `0`
    - VSD: Vendor Specific Data Block -> change the value of `DC_Y444` to `0`
4. Use the modified EDID file:
    1. Option -> Recalc Checksum
    2. File -> Save EDID Binary (to `~/modified_edid.bin` for example)
    3. Copy to firmware directory: `sudo install -Dm644 ~/modified_edid.bin /usr/lib/firmware/edid/modified_edid.bin`
    4. Configure kernel to use it. For example on Fedora: `sudo grubby --args='drm.edid_firmware=edid/modified_edid.bin' --update-kernel=DEFAULT`
        - Note that this only changes the current kernel so you can boot into older kernels (or edit in the grub menu) if it fails and revert this with `grubby --remove-args`
  5. Reboot
