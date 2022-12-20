+++
title = "Rojo wrangling"
date = 2022-12-19 12:00:00
+++

I have particular preferences about how I want my projects to be structured, and
I will bend my tools to make it work. This time, the tool being flexed is
[Rojo][rojo].

I want related modules to be grouped together into one "package". If a package
has a server component and a client component, I want those two files to live
next to each other under the same folder. However, there are two problems that
make this structure difficult to have.

Problem #1 is how Roblox handles replication. The client component has to be in
one location in order to replicate to clients, while the server component has to
be in a different location in order to be isolated from clients. They inherently
cannot be together (how dramatic).

There *is* the new [RunContext][RunContext] property that might solve this
problem, but I haven't explored its uses in full, and I'm somewhat skeptical of
its utility. More importantly, it doesn't have first-class support in Rojo, so
it's not terribly easy to use.

Problem #2 is that the default structure for Rojo projects is rather literal: a
file corresponds to an instance. With a simple tree definition, Rojo causes the
file structure to correspond mostly to the DataModel structure, which means
DataModel problems become file system problems.

There is an out, though. In Rojo, projects are recursive. While traversing the
project tree, if a `project.json` file is encountered, it will be turned into a
node by evaluating the content as a sort of sub-project. The rules for how this
works turns out to be very relaxed. Enough so that it's possible to get Rojo to
build just about any project structure if you put in the effort.

To generalize this concept, I introduce what I call "pointer files". These are
just regular `project.json` files, but they have barest minimum content:

```json
{"name":"NAME","tree":{"$path":"REFERENT"}}
```

Where `NAME` defines the name of the node, and `REFERENT` defines a path to the
file to be used as the node, relative to the project file. If you give each
pointer file a different name, then you can create any number of pointers in the
same folder.

## Packages example
As an example, let's say I have a `pkg` folder that I use to contain packages.
Each subfolder is one package, and "server" and "client" files within are the
respective components:

- `pkg/foo/server.lua`
- `pkg/foo/client.lua`
- `pkg/bar/server.lua`
- `pkg/bar/client.lua`

Then I have a separate `game` folder, which contains a literal representation of
the DataModel:

- `game/ServerScriptService`: Contains server components.
- `game/ReplicatedStorage`: Contains client components.

I can "unpack" my packages by creating a number of pointer files under the
`game` folder that point to files in the `pkg` folder:


```text
game/ServerScriptService/foo.project.json

    {"name":"foo","tree":{"$path":"../../pkg/foo/server.lua"}}

game/ServerScriptService/bar.project.json

    {"name":"bar","tree":{"$path":"../../pkg/bar/server.lua"}}

game/ReplicatedStorage/foo.project.json

    {"name":"foo","tree":{"$path":"../../pkg/foo/client.lua"}}

game/ReplicatedStorage/bar.project.json

    {"name":"bar","tree":{"$path":"../../pkg/bar/client.lua"}}
```

Finally, the root `default.project.json` points to the `game` folder, so that
building the project builds everything from there.

## Cloning example
This technique is surprisingly versatile. Here's another example: I have two
scripts that are used as the entrypoints for the server and client,
respectively. They both share a common "maid" module. The normal solution is to
have common modules stored under ReplicatedStorage. But I want the client
entrypoint to be snappy, so depending on modules outside of ReplicatedFirst is
not allowed. Instead, I have the structure set up as the following:

- `core/bootstrap.client/init.client.lua`
- `core/bootstrap.client/maid.project.json`
- `core/bootstrap.server/init.server.lua`
- `core/bootstrap.server/maid.project.json`
- `core/maid.lua`

Both `maid.project.json` files have the following content:

```json
{"name":"maid","tree":{"$path":"../maid.lua"}}
```

Then I have the usual pointers under the game tree to move the scripts to their
proper locations under ReplicatedFirst and ServerScriptService.

What's interesting is that, when Rojo builds the project, it creates a copy of
the `maid.lua` module under each bootstrap script. This allows me to have just
one file as the source of two separate modules! I'm sure this definitely wont
backfire in some subtle way!

## Automation
While my project is still in its infancy, I'm creating, removing, and renaming
files left and right. Manually keeping the pointer files up to date is an
exercise in futility, so I automate the whole thing with an [rbxmk][rbxmk]
script instead. This script defines how to map files around, while the
[Build.rbxmk.lua][Build.rbxmk.lua] library does the heavy lifting. An example
script might look like this:

```lua
-- Require the Build library.
local Build = rbxmk.runFile(path.expand("$sd/lib/Build.rbxmk.lua"))

-- Map package components to their respective locations.
Build.package("src/pkg", {
	boot = "game/ReplicatedFirst",
	server = "game/ServerScriptService",
	client = "game/ReplicatedStorage/client",
	shared = "game/ReplicatedStorage/shared",
	internal = "game/ReplicatedStorage/internal",
})

-- Map bootstrap scripts.
Build.ref("core/bootstrap.client", "game/ReplicatedFirst/bootstrap")
Build.ref("core/bootstrap.server", "game/ServerScriptService/bootstrap")

-- Remove any files that haven't been touched by this build script, which
-- accounts for renames/removals/etc.
Build.clean("game")
```

Unfortunately, the script requires the latest unreleased version of rbxmk, so
you'll have to build it yourself if you want to use this (sorry!). This post is
more to showcase the technique of abusing Rojo's project files to do crazy
things anyway.

This technique is very general, so there's nothing stopping you from
implementing it with your preferred method of automation. Come up with a
structure that best suits your needs!

[rojo]: https://rojo.space/
[RunContext]: https://robloxapi.github.io/ref/enum/RunContext.html
[rbxmk]: https://github.com/anaminus/rbxmk
[Build.rbxmk.lua]: https://gist.github.com/Anaminus/135999033fa01a3b1491b0d0e54b6f68