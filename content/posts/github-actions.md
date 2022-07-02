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

# Writing workflows

## Notifying your Discord Server using an existing action

```yml
name: Notify Discord
on:
  push:
    branches: [main]
jobs:
  notify-discord:
    name: Notify Discord
    runs-on: ubuntu-latest

    steps:
    - uses: sarisia/actions-status-discord@v1.9.0
      if: always()
      with:
        webhook: ${{secrets.DISCORD_WEBHOOK}}
        avatar_url: "https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png"
```

## Setting up a runner to generate a documentation site with DocFX

```yml
name: DocFX Build and Publish

on:
  push:
    branches: [ main ]
    
jobs:
  generate-docs:
    runs-on: windows-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Setup .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: 3.1.x
      - name: Setup DocFX
        uses: crazy-max/ghaction-chocolatey@v1
        with:
          args: install docfx
      - name: DocFX Build
        working-directory: Documentation
        run: docfx docfx.json
        continue-on-error: false
      - name: Publish
        if: github.event_name == 'push'
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: Documentation/_site
          force_orphan: true
```

## Setting up a runner and using bash scripts to build the game with Game-CI

```yml
name: Build Project
on:
  push:
    branches: [release]
  pull_request:
    branches: [release]
  workflow_dispatch:

jobs:
  build-standalone-windows-64:
    name: Build Standalone Windows 64
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - uses: actions/cache@v2
        with:
          path: Library
          key: Library-${{ hashFiles('Assets/**', 'Packages/**', 'ProjectSettings/**') }}
          restore-keys: |
            Library-

      - name: Build StandaloneWindows64
        uses: game-ci/unity-builder@v2
        id: build
        env:
          UNITY_EMAIL: ${{ secrets.UNITY_EMAIL }}
          UNITY_PASSWORD: ${{ secrets.UNITY_PASSWORD }}
          UNITY_SERIAL: ${{ secrets.UNITY_SERIAL }}
        with:
          targetPlatform: StandaloneWindows64
          versioning: Semantic

      - name: Clone public repo
        uses: actions/checkout@v3
        with:
          repository: WoahPieStudios/GDELECT4-ADVAPROD-BUILDS
          path: public-builds
          token: ${{ secrets.API_TOKEN_GITHUB }}

      - name: Compress and copy build to public repo
        run: |
          chmod +x .github/scripts/split-zipper.sh
          ./.github/scripts/split-zipper.sh ./build/StandaloneWindows64 100 public-builds/build/StandaloneWindows64/${{ steps.build.outputs.buildVersion }} ${{ steps.build.outputs.buildVersion }}

      - name: Push build to public repo
        run: |
          cd public-builds
          git config user.name bulletproofpancake
          git config user.email 57520402+bulletproofpancake@users.noreply.github.com
          git add .
          git commit -m "Update from https://github.com/${GITHUB_REPOSITORY}/commit/${GITHUB_SHA}"
          git push --force

      - name: Return License
        uses: game-ci/unity-return-license@v2
        if: always()
```