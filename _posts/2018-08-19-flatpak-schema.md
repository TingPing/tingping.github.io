---
layout: post
title: Easier Flatpak manifest editing with VSCode
tags: [flatpak]
---

As `flatpak-builder` has gained numerous features over the years it
is easy to not remember every single property and it becomes a bit
of a chore to keep running `man flatpak-manifest`.

<!-- more -->

So over a year ago I wrote a [JSON Schema](https://github.com/TingPing/flatpak-manifest-schema)
for manifests but after doing so I never ended up actually using it since I
found editor support to be lacking and Python libraries for validation gave
useless output when I tried to add support to GNOME-Builder.

Lately I've been using VSCode and it happens to expose some configuration
for schema files. The main problem we have is that flatpak manifests do not have
a custom file name or extension so all automatic schema association fails.
(I have proposed `.flatpakmanifest` in the past but nobody seems interested.)

### Setup

The most simple method to use a schema in VSCode is this custom property in the manifest itself:

```json
{
    "$schema": "https://raw.githubusercontent.com/TingPing/flatpak-manifest-schema/master/flatpak-manifest.schema"
}
```

If you want to keep the manifest clean of non-standard properties like that
you set these properties in your workspace (YAML support is from `redhat.vscode-yaml` extension) :

```json
{
    "json.schemas": [
        {
            "fileMatch": ["*.json"],
            "url": "https://raw.githubusercontent.com/TingPing/flatpak-manifest-schema/master/flatpak-manifest.schema"
        }
    ],
    "yaml.schemas": {
        "https://raw.githubusercontent.com/TingPing/flatpak-manifest-schema/master/flatpak-manifest.schema": "*.yml",
    }
}
```

### Results

<video width="900" controls loop autoplay>
  <source src="/video/json-schema-example.webm" type="video/webm">
</video>
