+++
title = "Managing assets locally"
description = "Keeping your Roblox assets on your computer."
date = 2022-11-23 12:00:00
tags = ["roblox"]
+++

If you're not indoctrinated into the cult of The Cloud™, then managing assets on
Roblox can be a pain. The assumption seems to be that you're meant to upload
everything and subject it to moderation whether it's ready or not.

I prefer to keep assets local for as long as possible. Roblox Studio has an
option to hot-reload assets that live on the local file system, which is
absolutely invaluable for fast iteration. Uploading a million variations of
textures that I'll never use ever again doesn't make any sense.

The problem with locally-sourced assets is that there aren't many locations that
Studio will read assets from. One of them is Roblox's content folder, which is
referred to using the `rbxasset://` scheme. This contains most of the assets
Roblox uses for their user interfaces, plugins, and whatnot. Unfortunately, it's
annoying for developers to use for their own assets, because the location
changes every time Roblox updates.

This could be worked around with some tooling, but Studio does have an
additional folder that is more persistent. It's located in the same place where
settings are stored. On Windows, this is the following:

```
%LocalAppData%\Roblox\LocalAssets
```

This folder behaves like the content folder. Files in here can be referred to
using the same `rbxasset://` scheme. Studio sort of merges this with the content
folder, checking the other folder if a file was not found in the first.

Studio also has a hidden setting that controls the location of this folder. In
the same place as LocalAssets (`%LocalAppData%\Roblox`), there is the
`GlobalSettings_13.xml` file (the numeric suffix may vary). Within this file is
the "Studio" class. Within the Studio class is a "LocalAssetsFolder" setting
that doesn't appear in the normal settings list. This setting determines which
folder local assets will be read from.

The problem with this folder is that it's only *one* folder. Us developers, we
tend to make many projects. And having to keep project-specific assets outside
of the project isn't fun to deal with.

My go-to solution is to use symbolic links, which is (very fortunately)
supported by Studio. The idea is to have a folder in your project that is used
for assets (e.g. `project/assets`). A link to this folder is then made in the
LocalAssets folder, using the project's name as the name of the link.

Windows has the `mklink` command for making symbolic links:

```batch
cd %LocalAppData%\Roblox\LocalAssets
mklink /D project path\to\project\assets
```

This will make a directory link called "project" in the LocalAssets folder. Now
if I have the texture `project/assets/foobar.png`, I can refer to it in Studio
as `rbxasset://project/foobar.png`.

I've simplified this process with a `.bat` script that makes it possible to drag
a project's asset folder into the script file, and the symbolic link is created
automatically:

```batch
@echo off
:: Replace with location configured by LocalAssetsFolder.
cd %LocalAppData%\Roblox\LocalAssets
for %%a in ("%~p1.") do set "x=%%~nxa"
mklink /D "%x%" "%1"
pause
```

Note that this assumes the asset folder is located in the root of the project.
The script looks at the name of the parent directory of the given folder to get
the project name, which is used to name the link.

As demonstrated, some tooling is required to manage assets locally, but at least
it only needs to be run once, at the start of a new project.

I'm still working out the best way to deal with turning local assets into The
Cloud™ assets. I have some ideas, but I'll save it for another post.
