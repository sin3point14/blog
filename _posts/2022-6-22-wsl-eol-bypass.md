---
layout: post
title: Bypassing the WSL EOL 31-10-2021 check
date: 2022-6-22
---

> I use an old version of Windows 10 and am in no mood to upgrade for some time.

When I try to run WSL in my Windows Terminal this is what I get:

![Windows Terminal]({{ site.baseurl }}/images/2022-6-22-wsl-eol-bypass/wt.png)

For the past 6 months I am manually changing system date, running WSL and restoring system date. It gets pretty annoying if I miss some step. For example, Firefox doesn't open any website with an incorrect system date and YouTube videos get stuck forever buffering.

My original plan was to reverse engineer wsl binaries+DLLs to figure out where this date check is but I couldn't find any leads. So I gave up and made a [small powershell wrapper](https://gist.github.com/sin3point14/f2ed9d0f064d8ad471c875db5e3d0a1e) that can be configured in Windows Terminal and handle the date changes.

Set your Command Line property for a Profile as:

```
powershell "<path/to/script> <distro-name>"
```

Here's mine:

![Windows Terminal Settings]({{ site.baseurl }}/images/2022-6-22-wsl-eol-bypass/wt-settings.png)

A thing which can be improved: The script spawns 2 admin powershells, meaning 2 UAC popups. That can probably be reduced to 1 but I'm not good enough at powershell to do that.
