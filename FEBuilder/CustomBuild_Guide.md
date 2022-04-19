## Introduction
There's been a fair amount of confusion regarding FEBuilder's skills system custom build function
and how to get it working. I was one of the few poeple crazy enough to make quite a few changes to it,
so I figured I'd write this guide to help people use custom build.

The first thing you're going to need is the [custom build folder.](https://dw.ngmansion.xyz/doku.php?id=en:en:guide:febuildergba:skillsystems_custombuild) **Do not use the buildfile version available on github. It is not compatible with custom build.**

## The Config File
Skillsystems has a lot of stuff on by default that you might not want such as leadership stars or the strength/magic split.  
Thankfully, some of them can be easily turned off in the config file located at `Root/EngineHacks/Config.event` (Root would be the folder custom build folder)  
Simply go to whatever `#define ...` line and put `//` before it to comment it out. The documentation in the config file tells you what each definition does.

For example going from:
```c
#define USE_STRMAG_SPLIT
```
to
```c
// #define USE_STRMAG_SPLIT
```

You can also do the opposite to enable previously disabled hacks.

<!--- Perhaps cover more advanced stuff like adding skills --->


## Running The Custom Build
<!--- Include screenshots. People seem to like those. --->
First, you'll need to have skillsystems installed on your FE8 ROM.  
Once you do that, go to the advanced menu, and click `Skill Custom Build`

In the new window, direct FEBuilder to `MAKE CUSTOM_BUILD.cmd` in your custom build folder,
and direct FEBuilder to a clean FE8 ROM.  
![](<Images/CustomBuild1.png>)  
![](<Images/CustomBuild2.png>)

<!--- Figure out skill takeover --->

Click `Build Started`
