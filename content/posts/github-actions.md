---
title: "Automate your game builds in Github with GameCI and Github Actions"
date: 2022-06-18T09:39:15+08:00
draft: true
---

Building Unity projects are relatively quick and consistent, but most of the time this renders your computer unusable until it is complete, thankfully we have [GameCI](https://game.ci) to the rescue.

> As always, to get a better understanding of the tools presented in this article, I recommend reading the documentation for each tool if available.

# Setting up Github Actions
First and foremost, I have to state that if you want to use Github Actions for free, it is important that you use it on a **public** repository. By default, Github's free tier [offers 2,000 minutes a month](https://github.com/pricing) (which I believe is tracked for **all** of your private repositories for that month) but is free for public repositories. Despite this limit, I still believe it is beneficial to delegate building projects to another computer as it allows you to still use your computer while the game is being built especially if you are going to be building for multiple platforms. If you have a spare computer lying around, you can use it as a [self-hosted runner](https://docs.github.com/en/enterprise-server@3.2/actions/hosting-your-own-runners/about-self-hosted-runners) which does [not get counted](https://github.community/t/does-using-self-hosted-runners-add-to-action-minutes-usage/18281) when used in a private repository. **So if you think you are going to reach that 2,000 minute cap and your assets can be redistributed, go public. If you have proprietary or paid assets that are not allowed to be redistributed, either go private and automate the build sparingly or use self-hosted runners.**

## Writing the workflow file