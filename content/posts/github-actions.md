---
title: "Automate your game builds in Github with GameCI and Github Actions"
date: 2022-06-18T09:39:15+08:00
draft: true
---

Building Unity projects are relatively quick and consistent, but most of the time this renders your computer unusable until it is complete, thankfully we have [GameCI](https://game.ci) to the rescue. For this article I will showcase how I used GameCI and Github Actions in order to help me and my teammates develop our game [Get Guns, Go Grapple](https://www.github.com/WoahpieStudios/GDELECT4-ADVAPROD).

> As always, to get a better understanding of the tools presented in this article, I recommend reading the documentation for each tool if available.

# Setting up Github Actions
First and foremost, I have to state that if you want to use Github Actions for free, it is important that you use it on a **public** repository. By default, Github's free tier [offers 2,000 minutes a month](https://github.com/pricing) (which I believe is tracked for **all** of your private repositories for that month) but is free for public repositories. Despite this limit, I still believe it is beneficial to delegate building projects to another computer as it allows you to still use your computer while the game is being built especially if you are going to be building for multiple platforms. If you have a spare computer lying around, you can use it as a [self-hosted runner](https://docs.github.com/en/enterprise-server@3.2/actions/hosting-your-own-runners/about-self-hosted-runners) which does [not get counted](https://github.community/t/does-using-self-hosted-runners-add-to-action-minutes-usage/18281) when used in a private repository. **So if you think you are going to reach that 2,000 minute cap and your assets can be redistributed, go public. If you have proprietary or paid assets that are not allowed to be redistributed, either go private and automate the build sparingly or use self-hosted runners.**

## Writing the workflow file

In order to use Github Actions, you would need to create a workflow file in the root of your repository. Workflows need to be at a specific directory in order to work so at the root of your project repository create a folder and name it `.github` then create a folder inside that and name it `workflows`. Inside this workflows directory, we can then finally create a workflow file, I have it named as `build-project.yml`

<!-- Insert photo of directory here -->
### Building the game

Opening`build-project.yml` in a text editor  would show the following:

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
      
      - name: Compress Build
        uses: edgarrc/action-7z@v1
        with:
          args: 7z a -t7z -mx=9 ${{ steps.build.outputs.buildVersion }}.7z ./build/StandaloneWindows64/*

      - name: Clone public repo
        uses: actions/checkout@v3
        with:
          repository: WoahPieStudios/GDELECT4-ADVAPROD-BUILDS
          path: public-builds
          token: ${{ secrets.API_TOKEN_GITHUB }}
        
      - name: Copy build to public repo
        run: cp ${{ steps.build.outputs.buildVersion }}.7z public-builds/build/StandaloneWindows64/${{ steps.build.outputs.buildVersion }}.7z

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
now let us break it down.

#### Breaking it down

The `name` parameter is essentially how we identify the workflow in Github. In this case it is named as `Build Project`.

```yml
name: Build Project
```

The `on` parameters identify when this workflow would be ran. The `branches` parameter identify which branches this workflow would get triggered. In this case, it is the `release` branch. The `push` and     `pull request` parameters mean that when we push or create a pull request to the release branch, we would run this workflow. The `workflow_dispatch` parameter enables us to run the workflow manually in the Github Actions page and select any branch or tag to run the workflow on.

```yml
on:
  push:
    branches: [release]
  pull_request:
    branches: [release]
  workflow_dispatch:
```

The `jobs` parameter is where we would implement Game CI and other integrations. These jobs can be done sequentially or in parallel to each other, but doing them in parallel would bring the need for more runners so keep this in mind if you are self-hosting. 