---
title: "Resharper Clt"
date: 2022-07-29T17:58:59+08:00
draft: true
---

# InspectCode tool
1. Alias Unity in PowerShell
2. Ensure that the JetBrains Rider Editor package is in the project
3. Generate the project files by running `Unity -batchmode -quit -projectPath ProjectPath -executeMethod Packages.Rider.Editor.RiderScriptEditor.SyncSolution`
4. Download the [ReSharper Command Line Tools for dotnet](https://www.jetbrains.com/help/resharper/ReSharper_Command_Line_Tools.html#install-and-use-resharper-command-line-tools-as-net-core-tools)
5. Run `dotnet jb inspectcode .\ParkingRSSI-Unity.sln -x="JetBrains.Unity" -f="Html" -o="CodeIssues.html"`