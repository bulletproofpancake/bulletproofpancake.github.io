---
title: "Automate your development with Github Actions"
date: 2022-06-26T22:50:50+08:00
draft: true
---

Building projects are pretty simple, most of the time you just press a shortcut, wait for a bit, check the build folder, zip it and upload it somewhere so others can get it. But what if you would need to create builds for different platforms, do you do them sequentially on your computer? What about the storage? Surely, you can't keep every build you've made, especially if there's multiple platforms involved, even if you can, how can you make sure that it is properly versioned and organized? What about documentation? Wouldn't it be nice if there's something that can just pull all the comments from your code and wrap it in a nice little bow? Thankfully, I've found a solution:

![](https://i.imgur.com/tm5ohhh.png)

# Github Actions

So what is Github Actions anyway? Basically, it is a set of instructions that gets run on a computer depending on what has happened in your repository. You can [generate a documentation site for your code](https://dotnet.github.io/docfx/), [run shell scripts to use with your project](https://github.community/t/running-a-bash-script/141584), or even [make a build of your project](https://game.ci/docs/), and that is just what I used during the development of our thesis game [Get Guns, Go Grapple](https://github.com/WoahPieStudios/GDELECT4-ADVAPROD). There are a lot more things that you can do with it, for example: [this site is being made available to you thanks to Github Actions](https://github.com/bulletproofpancake/bulletproofpancake.github.io/actions). The possibilities are endless provided you know the tools available to you.

I would like to preface that while [Github Actions is free for public repositories (labeled as CI/CD minutes)](https://github.com/pricing) it can still be used in private repositories by [self-hosting runners](https://docs.github.com/en/enterprise-server@3.2/actions/hosting-your-own-runners/about-self-hosted-runners) and it would [not get counted as used minutes](https://github.community/t/does-using-self-hosted-runners-add-to-action-minutes-usage/18281), however this would require you having another computer available to host a runner.

> As always, I would recommend reading the documentation about Github Actions [here](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions) for a more in-depth explanation as this article would mostly be a showcase of how I used it during the development of *Get Guns, Go Grapple*.


# Writing workflows

Workflows contain the details of your automation process. They contain the conditions when they will run, what system they would use, and the steps of the automation. They are YAML files and placed in the `.github/workflows/` directory that you can create at the root of your repository.

The following is an example of a workflow file:

```yml
# This is a basic workflow to help you get started with Actions

name: Demo Workflow

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the $default-branch branch
  push:
    branches: [ $default-branch ]
  pull_request:
    branches: [ $default-branch ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      # Runs a single command using the runners shell
      - name: Run a one-line script
        run: echo Hello, world!

      # Runs a set of commands using the runners shell
      - name: Run a multi-line script
        run: |
          echo Add other actions to build,
          echo test, and deploy your project.
```

## Generating a documentation site with DocFX
[DocFX](https://dotnet.github.io/docfx/) is a static documentation generator. This means that it generates source code from the project and creates a site for it which can be uploaded to a server. As this article focuses on the utilization of Github Actions, I would recommend reading [Normand Erwan's guide for using DocFX with Unity](https://normanderwan.github.io/DocFxForUnity/) which I have referenced extensively for this. For this example, I used [Github Pages](https://pages.github.com) to host the documentation site and it is pretty easy to setup.

Once DocFX is set up in your project repository, we can now setup our workflow file:
```yml
name: DocFX Build and Publish
on:
  # Triggers the workflow when commits are pushed to main
  push:
    branches: [ main ]
    
jobs:
  generate-docs:
    # uses a windows machine to run the job
    runs-on: windows-latest
    
    steps:
      # Clones the repository to the workspace
      - name: Checkout
        uses: actions/checkout@v2
      
      # Installs .NET Core which is used by DocFXs
      - name: Setup .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: 3.1.x
      
      # Uses the chocolatey package manager to install DocFX
      - name: Setup DocFX
        uses: crazy-max/ghaction-chocolatey@v1
        with:
          args: install docfx
      
      # Uses DocFX to build the documentation site
      - name: DocFX Build
        working-directory: Documentation
        run: docfx docfx.json
        continue-on-error: false
      
      # Publishes the built site on Github Pages
      - name: Publish
        if: github.event_name == 'push'
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: Documentation/_site
          force_orphan: true
```
In order to access the documentation site, go to `{username}.github.io/{repository-name}`. If the site returns a 404 error, ensure that the site is being published by going to your repository settings, select Pages, and ensure that the branch `gh-pages` is being used.

![](https://i.imgur.com/UpKR6Ft.gif)

The reason for this is that unlike our `main` branch or the other branches we make in our repository, the `gh-pages` branch is generated by the action and contains the necessary files for the site, whereas the other branches contain our project files.

![](https://i.imgur.com/2CcyUTw.gif)

## Notifying your Discord Server

During development our team has chosen Discord as our means of communication for the project. Given that I instructed the team to only create branches from main to ensure that everyone has the latest changes, I found myself still creating branches from an outdated version of main because I was unaware that there has been new changes in remote. Thankfully, setting up a bot in Discord to notify a channel whenever main is updated is easy. For this, I used [sarisia's action](https://github.com/marketplace/actions/actions-status-discord) which only needs a Discord Webhook in the repository environment.

To create a Discord Webhook:
1. Create a channel where you would send your messages to, and then edit its settings.
    
    ![](https://i.imgur.com/C2EHOxD.gif)

2. Select the Integrations tab
    
    ![](https://i.imgur.com/laZNTzG.gif)

3. Create a webhook and copy its url

    ![](https://i.imgur.com/RVYb3z5.gif)

4. In your Github repository, go to Settings > Secrets > Actions

    ![](https://i.imgur.com/7EeqPtG.gif)

5. Create a new repository secret, name it `DISCORD_WEBHOOK` and paste the webhook url into the value field
   
    ![](https://i.imgur.com/pk4hDPy.gif)

Once that is done, we can now write our workflow file:

> You can just paste this into your repository as long as you remember to place it in the `.github/workflows/` directory.

```yml
name: Notify Discord
on:
  # Triggers the workflow when commits are pushed to main
  push:
    branches: [main]

jobs:
  notify-discord:
    name: Notify Discord
    runs-on: ubuntu-latest

    steps:
      # The action used to send notifications to Discord
    - uses: sarisia/actions-status-discord@v1.9.0
      # Always sends a notification to the channel
      if: always()
      with:
        # A webhook linked to the Discord Channel
        webhook: ${{secrets.DISCORD_WEBHOOK}}
        # Changes the profile picture to Octocat
        avatar_url: "https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png"
```

If you have setup everything correctly, every time you push something to the `main` branch you should receive a notification from your bot like so:

![](https://i.imgur.com/fkpRSjI.png)

## Building and Deploying the game with Game-CI

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