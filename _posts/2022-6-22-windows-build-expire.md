---
layout: post
title: Removing "The build of Windows will expire soon" popups
date: 2022-6-22
---

I use an old version of Windows 10 and am in no mood to upgrade for some time. This annoying popup shows up around every 15 mins and I decided to get rid of it:

![Annoying screen]({{ site.baseurl }}/images/2022-6-22-windows-build-expire/annoying.png)  

This has worked fine for me till now
Steps:

- Locate C:\Windows\System32\LicensingUI.exe
- Take ownership of the binary from TrustedInstaller to your user and add permissions to Modify to the Users group

![Permissions]({{ site.baseurl }}/images/2022-6-22-windows-build-expire/perms.png)  

- Rename the binary to something else.  
- \[optional\] Restore permissions on the binary

### How I found this?

I used [this script](https://gist.github.com/sin3point14/52a3404cbbbfcf51361351227a4e6099) to find the process ID for this process by hovering my mouse over the popup GUI.  
Then used Task Manager > Details to locate the executable for this PID.

The Windows developers must have taken care in their code to handle this process executable not being able to launch. So I took the liberty to ensure it never launches :p.
