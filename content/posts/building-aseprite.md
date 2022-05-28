---
title: "Building Aseprite from source without using Visual Studio for Windows"
date: 2022-05-12T20:56:13+08:00
taxonomies:
    category: guide
    tag: aseprite, CLI, pixel-art
---

[Aseprite](https://www.aseprite.org) is a pixel-art tool to create 2D animations, sprites, and any kind of graphics for games. Aseprite usually costs 449.95 pesos on [Steam](https://store.steampowered.com/app/431730/Aseprite/), but it is actually free if you build it yourself from [source](https://github.com/aseprite/aseprite).

I will mostly be following their [install guide](https://github.com/aseprite/aseprite/blob/main/INSTALL.md), and while this is good, I found it confusing the first time so I hope this will help streamline the process.

# Dependencies
To compile Aseprite on Windows, you would need the following:
- [CMake](https://cmake.org) (3.16 or greater)
- [Ninja](https://ninja-build.org) build system
- A prebuilt package of the [Skia-m102](https://github.com/aseprite/skia/releases) library
- Desktop development with C++ item + Windows 10.0.18362.0 SDK from the [Visual Studio Installer](https://visualstudio.microsoft.com/downloads/?q=build+tools#build-tools-for-visual-studio-2022)

## Getting the dependencies

### CMake and Ninja

For CMake and Ninja, I prefer to install both of them through a package manager like [scoop](https://scoop.sh) or [chocolatey](https://chocolatey.org) as I have had issues when I installed and downloaded them individually before. I would be using scoop for this tutorial.

Inside a terminal, run
```
scoop install cmake; scoop install ninja
```
in order to install CMake and ninja in a single command.

![](https://i.imgur.com/hP7OHK6.gif)

### Skia

For Skia, get the `m102` package from [here](https://github.com/aseprite/skia/releases) and download the Windows release. Once it has been downloaded, extract it to a folder, rename it to `skia`, and transfer it to `C:\deps\` for the install later.

![](https://i.imgur.com/LLK100M.gif)

### Visual Studio Installer

Grab the Visual Studio Installer [here](https://visualstudio.microsoft.com/downloads/?q=build+tools#build-tools-for-visual-studio-2022) and select Build Tools for Visual Studio 2022. Once the Visual Studio Installer has been installed to your system, select the Desktop development with C++ and the Windows 10.0.18362.0 SDK from the optional items.

![](https://i.imgur.com/xTqUYbZ.gif)

# Getting the source

The source code for Aseprite is available on their [Github](https://github.com/aseprite/aseprite/releases).