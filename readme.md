
Userspace containers for building the Gimp application.

!!! Work in progress. Much doesn't work.
I have long used similar scripts, and this repo is partly just gathering them together.

The containers are for *developing and testing* Gimp.
They let you edit the Gimp source and build.
There are many containers so you can build in different environments,
i.e. with varied Linux distributions, compilers, and libraries.

Everything is scripted and so repeatable.
Only the scripts are in this repo.
All dependencies are installed into a container.
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
The advantage of vagga is that you you never (hardly ever) need root privilege.

# Status

Don't expect everything to work out-of-the box.
This repo may often change, and the things it references may fall out of date.
It can just document something that worked once.

# Quick start

```
install vagga
clone this repo
cd alpineMesonGcc  (that is the "work" directory)
git clone https://gitlab.gnome.org/GNOME/gimp.git (get the source into the work directory)
vagga --use-env DISPLAY run
```

That should build Gimp and run it, in a container.
Note you haven't installed anything but vagga on your machine.

To delete the container:

```
rm -rf .vagga
```

The container is all in .vagga.

## Vagga userspace containers

The vagga tool creates userspace containers.
This means you do not need root privileges or sudo.
You can simply delete a .vagga directory to remove containers.
Your base system is untouched.

A container is just a rooted file system, related to the chroot command.

## About our use of vagga

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

TODO

## Configuring vagga

You will want to configure the vagga cache.

TODO

## About the container structure

The containers are stacked (inherited) one on top of the other
(done using vagga capabilities which uses links.)
This is done simply as a matter of documentation,
but also so that changing one container need not rebuild one huge container.

For example, starting with the base container, the stacked containers hold:

    - the OS with some tools.
    - more tools specific to a Gimp build.
    - packaged libraries required by Gimp
    - built libraries required by Gimp (gegl and babl)

## Build configurations

Each subdirectory builds and runs the Gimp app using different tools.
The name of a subdirectory denotes the build system and compiler.  Unless otherwise stated the OS is the latest Ubuntu.

The choices are:

    - alpineMesonGcc : build under the alpine distribution using the meson build system and the gcc compiler

    - mesonClang : build under Ubuntu distribution using meson and clang

    - automakeGcc : build under Ubuntu distribution using automake and gcc

    - valgrindPlugin: sets up the environment for Gimp to valgrind a plugin

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

Run the "run" command as above.
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

(Repos are cheap.  
The scripts expect a separate gimp repo in each work directory.
That doesn't make it easy to build the same repo in different ways.
FIXME: share the gimp repo among many vagga scripts.
)

## Basic usage

vagga --use-env DISPLAY run

That builds and runs the GIMP app, using the display (windows) of your system.

"run" is a command, named by vagga conventions.
Here it always means: build, install, and run Gimp.

## Finding your way around the container

Not in the container, but in the work directory:

    - the gimp repo (the source code you can edit)
    - the build directory 
    - .home, your HOME directory while in the container

In the container (somewhere inside the .vagga directory)

    - installed gimp
    - built and installed babl and gegl libraries

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

## Debugging

You can edit the command portion of the vagga.yaml script
to run the debugger (dbg) or valgrind, ahead of Gimp.

## Advanced development

You can edit the container portions of the vagga.yaml script to install different OS, tools, or packaged dependencies.

You can also edit the script to build the very latest libraries that Gimp depends on, instead of using older versions packaged in a distribution.

## Finally, cleaning up

Don't forget to commit and push any changes to the gimp repo in the work directory!!!
My origin is a personal gimp clone on gitlab.gnome.org.
I push to that origin, and submit MR's from there.

When you are done, you can simply delete the containers (in a hidden file):

```
>rm -rf .vagga  
```

Sometimes you can also free up old containers by:

```
vagga _clean --unused
```

You can also delete the build directory.
