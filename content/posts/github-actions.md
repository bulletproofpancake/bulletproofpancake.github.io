---
title: "Automate your development with Github Actions"
date: 2022-06-26T22:50:50+08:00
draft: true
---

Building projects are pretty simple, most of the time you just press a shortcut, wait for a bit, check the build folder, zip it and upload it somewhere so others can get it. But what if you would need to create builds for different platforms, do you do them sequentially on your computer? What about the storage? Surely, you can't keep every build you've made, especially if there's multiple platforms involved, even if you can, how can you make sure that it is properly versioned and organized? What about documentation? Wouldn't it be nice if there's something that can just pull all the comments from your code and wrap it in a nice little bow? Thankfully, I've found a solution:

![](https://i.imgur.com/tm5ohhh.png)

# Github Actions

So what is Github Actions anyway? Basically, it is a set of instructions that gets run on a computer depending on what has happened in your repository. You can [generate a documentation site for your code](https://dotnet.github.io/docfx/), [run shell scripts to use with your project](https://github.community/t/running-a-bash-script/141584), or even [make a build of your project](https://game.ci/docs/), and that is just what I used during the development of our thesis game [Get Guns, Go Grapple](https://github.com/WoahPieStudios/GDELECT4-ADVAPROD). There are a lot more things that you can do with it, for example: [this site is being made available to you thanks to Github Actions](https://github.com/bulletproofpancake/bulletproofpancake.github.io/actions). The possibilities are endless provided you know the tools available to you.

> I would like to preface that while [Github Actions is free for public repositories (labeled as CI/CD minutes)](https://github.com/pricing) it can still be used in private repositories by [self-hosting runners](https://docs.github.com/en/enterprise-server@3.2/actions/hosting-your-own-runners/about-self-hosted-runners) and it would [not get counted as used minutes](https://github.community/t/does-using-self-hosted-runners-add-to-action-minutes-usage/18281), however this would require you having another computer available to host a runner.

## Building the game with Game-CI



## Generating a documentation site with DocFX