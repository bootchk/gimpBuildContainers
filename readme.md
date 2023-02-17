
Userspace containers for building the Gimp application.

!!! Work in progress. Much doesn't work.
I have long used similar scripts, and this repo is partly just gathering them together.

The containers are for *developing and testing* Gimp.
They let you edit the Gimp source and build.
There are many containers so you can build in different environments,
i.e. with varied Linux distributions, compilers, and libraries.

Everything is scripted and so repeatable.
Only the scripts are in this repo.
All tools and dependencies are installed into a container.
You use a command line, but commands are from the scripts,
and execute in the container.

Gimp is buit in containers, so your development system is untouched,
and yet it seems as if using many development systems and target systems.

There are many other scripted builds of gimp.
The difference here is that there are *many, different* scripted builds.
And the difference is that the scripts are vagga scripts,
not shell or Python scripts.
Unfortunately, you may need to learn vagga.

Alternatively, you can use other container scripts, such as Docker scripts, or NixOS?
The advantage of vagga is that you never (hardly ever) need root privilege.

# Status

Don't expect everything to work out-of-the box.
This repo may often change, and the things it references may fall out of date.
It can just document something that worked once.

# Quick start

```
install vagga
clone this repo
cd mesonClang  (that is the "work" directory)
git clone https://gitlab.gnome.org/GNOME/gimp.git (get the source into the work directory)
vagga --use-env DISPLAY run
```

That should build Gimp and run it, in a container, using the display of the host.
Note you haven't installed anything but vagga on your machine.

Touch something in the gimp repo, and repeat the last command.
It should rebuild (compile and link) just what needs to be rebuilt, and reinstall Gimp in the container.

To delete the container, which is all in one directory:

```
rm -rf .vagga
```

## Vagga userspace containers

The vagga tool creates userspace containers.
This means you do not need root privileges or sudo.
You can simply delete a .vagga directory to remove containers.
Your base system is untouched.

A container is just a rooted file system, related to the chroot command.

## About the use of vagga

Vagga seems to be commonly used to build containers for working programs.
That is, a tool for building system images, usually installed.

Here, we build a container for developing i.e. programming Gimp:
the container contains tools and environment.
The commands in the container build Gimp.

## Prerequisites

The containers are vagga containers.
You must first install vagga.
That is the only thing you need to install in your system.
Everything else is installed in a container and is easily deleted.

Follow the instructions here, for Ubuntu: https://vagga.readthedocs.io/en/latest/installation.html

vagga has one dependency, it requires newuidmap from package uidmap

```
>sudo apt install uidmap
```

You also need to install git.
Otherwise, your system doesn't need to have any developer tools,
since they will be installed in a container.

## Configuring vagga

For performance, you will want to configure the vagga cache.
This makes vagga cache the downloaded packages that a vagga script installs.
Since you probably will rebuild the containers sometime,
and since you probably will have many project (that is, containers)
you want a global cache for vagga.

See https://vagga.readthedocs.io/en/latest/tips.html which says:

Edit the settings file:

```
vi ~/.config/vagga/settings.yaml
```

and add this line:

```
cache-dir: ~/.cache/vagga/cache
```

Also, make the cache dir:

```
mkdir ~/.cache/vagga/cache
```

## About the container structure

The scripts create stacked containers (inherited) one on top of the other
(done using vagga capabilities which uses links.)
This is done simply as a matter of documentation,
but also so that changing one container need not rebuild one huge container.

For example, starting with the base container, the stacked containers typically hold:

    - the OS with some tools.
    - more tools specific to a Gimp build.
    - packaged libraries required by Gimp
    - built libraries required by Gimp (gegl and babl)

## Build configurations

Each subdirectory builds and runs the Gimp app using different tools.
The name of a subdirectory may denote the OS, build system, and compiler.  
Unless otherwise stated the OS is the latest Ubuntu.
Unless otherwise stated building 2.99 latest GIMP.

Example choices that work are:

    - mesonClang : build using meson and clang 

    - automakeGcc : build using automake and gcc
    
    - simpleMeson : build using meson and Gcc, without ANY env variables set

    - 2.10automakeGcc: build latest 2.10 using automake and Gcc (but doesn't build Python2 for GIMP)

Example choices that don't work (in progress) are:

    - alpineMesonGcc : build under the alpine distribution using the meson build system and the gcc compiler

    - valgrindPlugin: sets up the environment to valgrind a GIMP plugin

    - crossroadWindows: cross compiles on Linux for Windows (you can't run in it, it doesn't put Wine in the container.)

You must cd into a subdirectory.
The subdirectories are the "work" directories.
Each is independent.

### More about the alpine build

Vagga itself favors alpine.

Alpine OS is very lightweight and installs fast.

The names of the packages in alpine are different than in Ubuntu.

## The basic work flow

Change dir to a work directory.

Clone a gimp repo into it.

Edit the gimp source.

Run the "vagga --use-env DISPLAY run" command.
(The first time, this takes a long time, building the containers.)

Exercise the Gimp app.

Repeat (it will go faster now, only rebuilding the Gimp source you touched.)

## Setup a work directory

Clone a gimp repo into the work directory.
The vagga scripts expect a directory named gimp.
In the repo, you can checkout any version of Gimp.
The usual clone will get the "master" i.e. under development, version.

```
git clone https://gitlab.gnome.org/GNOME/gimp.git
```

You may already have your personal clone of Gimp at Gitlab.
Then you clone it, develop a branch, push to the origin, and create MR's to the main Gimp project.
Then your upstream is the main Gimp repo, your origin is your personal clone at Gitlab,
and you have a local clone in your work directories.

```
git clone https://gitlab.gnome.org/GNOME/<yourname>/gimp.git
```

(Repos are cheap.  
The scripts expect a separate gimp repo in each work directory.
That doesn't make it easy to build the same repo in different containers.
FIXME: share the gimp repo among many vagga scripts/containers.
)

## Basic usage

vagga --use-env DISPLAY run

That builds and runs the GIMP app, using the display (windows) of your host system.

"run" is a command, named by vagga conventions.
Here it always means: build, install, and run Gimp.

## Finding your way around the container

Not in the container, but in the work directory:

    - the gimp repo (the source code you can edit)
    - the build directory (e.g. gimpMesonClangBuild, an out-of-tree build)
    - .home, your HOME directory while in the container

In the container (somewhere inside the .vagga directory)

    - installed gimp
    - built and installed babl and gegl libraries
    - the installed dependencies (libraries) of gimp

the container is writeable just so that a build can install into the container
(see "write-transient" in the vagga script.)


## Other commands

Some vagga.yaml scripts may contain other scripts:

    - for querying attributes of the container.
    - starting a shell in the container
    - installing Gimp plugins in the container's home directory

Any command you want to execute in the container must be via a vagga command in the vagga.yaml script.

## Transient containers

The containers are not read-only, so that you can install the built Gimp into the container.
A build is in the container, but the artifacts are in the work directory e.g. in a mesonBuild dir.
Gimp is installed into the container.
See vagga docs about transient.

## Debugging Gimp in a container

You can edit the command portion of the vagga.yaml script
to run the debugger (dbg) or valgrind, ahead of Gimp.

You can also edit the command portion of the vagga.yaml script
to set environment variables which make Gimp verbose with debug messages.

G_DEBUG
GTK_DEBUG
GIMP_DEBUG
GIMP_PLUGIN_DEBUG

See examples in the scripts.

## Fresh install of Gimp

When a script installs Gimp into a container,
it uses the .home directory in the work directory,
as the home directory for the user.
This is outside the container.

Gimp stores its setting there.
A subsequent install of Gimp (running vagga again)
does not overwrite the config.

To get a fresh install, simply delete .home/.config/GIMP directory.

## Installing Gimp plugins

The plugins distributed with Gimp are installed into the container.

You can install your own plugins into the .home/.config/GIMP/... directories,
outside the container.
Some of the vagga scripts have example commands that do that.

## Advanced development

You can edit the container portions of the vagga.yaml script to install different OS, tools, or packaged dependencies.

You can also edit the script to build the very latest libraries that Gimp depends on, instead of using older versions packaged in a distribution.

## Finally, cleaning up

Don't forget to commit and push any changes you made in the gimp repo in the work directory!!!

When you are done, you can simply delete the containers (in a hidden file):

```
>rm -rf .vagga  
```

When you are running low on file space, 
you can also free up old containers by:

```
vagga _clean --unused
```

You can also delete the build directory and the .home directory.
So the work directory is back to just the vagga.yaml script.
Read the .gitignore file to see that it ignores
the same files, that you can delete.
