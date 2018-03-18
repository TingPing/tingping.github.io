---
layout: post
title: Flatpaking application plugins
tags: [linux flatpak]
---

Sometimes you simply do not want to bundle everything in a single package such
as optional plugins with large dependencies or third party plugins that are not
supported. In this post I'll show you how to handle this with Flatpak using
HexChat as an example.

<!--more-->

Flatpak has a feature called extensions that allows a package to be mounted within
another package. This is used in a variety of ways but it can be used by any
application as a way to insert any optional bits. So lets see how to define one (details omitted for brevity):

```json
{
  "app-id": "io.github.Hexchat",
  "add-extensions": {
    "io.github.Hexchat.Plugin": {
      "version": "2",
      "directory": "extensions",
      "add-ld-path": "lib",
      "merge-dirs": "lib/hexchat/plugins",
      "subdirectories": true,
      "no-autodownload": true,
      "autodelete": true
    }
  },
  "modules": [
    {
      "name": "hexchat",
      "post-install": [
       "install -d /app/extensions"
      ]
    }
  ]
}
```

The exact details of these are best documented in the Extension section of `man flatpak-metadata`
but I'll go over the ones used here:

- `io.github.Hexchat.Plugin` is the name of the extension point and all extensions will have the same prefix. 
- `version` allows you to have parallel installations of extensions if you break ABI or API for example, `2` refers to HexChat 2.x incase it makes a `3.0` with an API break (It is probably smart in the future to add the runtime version here also since it will break ABI also).
- `directory` sets a subdirectory where everything is mounted relative to your prefix so `/app/extensions` is
where they will go.
- `subdirectories` allows you to have multiple extensions and each one will get their own subdirectory. So `io.github.Hexchat.Plugin.Perl` is mounted at `/app/extensions/Perl`.
- `merge-dirs` will merge the contents of subdirectories that match these paths (relative to their prefix). So for this case the contents of `/app/extensions/Perl/lib/hexchat/plugins` and `/app/extensions/Python/lib/hexchat/plugins` will both be in `/app/extensions/lib/hexchat/plugins`. This allows limiting the complexity of your loader to only need to look in one directory (applications will need to be configured/patched to look there).
- `add-ld-path` adds a path, relative to extensions prefix, to the library path so for example `/app/extensions/Python/lib/libpython.so` can be loaded.
- `no-autodownload` will not automatically install all extensions which is the default.
- `autodelete` will remove all extensions when the application is removed.

So now that we defined an extension point lets make an extension:

```json
{
  "id": "io.github.Hexchat.Plugin.Perl",
  "branch": "2",
  "runtime": "io.github.Hexchat",
  "runtime-version": "stable",
  "sdk": "org.gnome.Sdk//3.26",
  "build-extension": true,
  "separate-locales": false,
  "appstream-compose": false,
  "build-options": {
    "env": {
      "PATH": "/app/extensions/Perl/bin:/app/bin:/usr/bin"
    }
  },
  "modules": [
    {
      "name": "perl"
    },
    {
      "name": "hexchat-perl",
      "post-install": [
        "install -Dm644 plugins/perl/perl.so ${FLATPAK_DEST}/lib/hexchat/plugins/perl.so",
        "install -Dm644 --target-directory=${FLATPAK_DEST}/share/metainfo data/misc/io.github.Hexchat.Plugin.Perl.metainfo.xml",
        "appstream-compose --basename=io.github.Hexchat.Plugin.Perl --prefix=${FLATPAK_DEST} --origin=flatpak io.github.Hexchat.Plugin.Perl"
      ]
    }
  ]
}
```

So again going over some key points quickly: `id` has the correct prefix, `branch` refers to the extension version,
`build-extension` should be obvious, `runtime` is what defines the extension-point. Some less obvious things to make note of is that your extensions prefix will not be in `$PATH` or `$PKG_CONFIG_PATH` by default
so you may need to set them (see `build-options` in `man flatpak-manifest`). `$FLATPAK_DEST` is also
defined as your extensions prefix though not everything expands variables.

While not required you also should install appstream metainfo for easy discover-ability. For example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component type="addon">
  <id>io.github.Hexchat.Plugin.Perl</id>
  <extends>io.github.Hexchat.desktop</extends>
  <name>Perl Plugin</name>
  <summary>Provides a scripting interface in Perl</summary>
  <url type="homepage">https://hexchat.github.io/</url>
  <project_license>GPL-2.0+</project_license>
  <metadata_license>CC0-1.0</metadata_license>
  <update_contact>tingping_AT_fedoraproject.org</update_contact>
</component>
```

Which will be shown in GNOME-Software:

![GNOME-Software](https://user-images.githubusercontent.com/798838/37436042-70287b82-27bc-11e8-8aa9-bea769eda444.png)
