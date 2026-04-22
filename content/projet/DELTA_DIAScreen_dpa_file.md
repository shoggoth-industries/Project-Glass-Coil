---
title: "DELTA DIAScreen file password bypass (.dpa file)"
description: "Monitor your ICS systems"
showDate: false
showReadingTime: false
---

Band-Aid on a broken arm: DIAScreen file password bypass (.dpa file)
====================================================================

Foreword
--------

This article is part of a series of articles focused on reverse engineering EWS software to determine the security of project files and potentially bypass the security measures applied to them.

### !!!!!!!!!! for educational purpose only !!!!!!!!!!!!!!!!!!!!!!!!!!!!!

Tools
-----

*   [ScyllaHyde](https://github.com/x64dbg/ScyllaHide)
*   [X32 dbg](https://x64dbg.com/)

Introductions
-------------

### Delta

Delta Electronics, Inc. (Chinese: 台達電子; also known as DELTA or Delta Electronics) is a Taiwanese electronics manufacturing company. Its headquarters are in Neihu, Taipei. It is known for its DC industrial and computer fans, data center rectifiers and switching power supplies. The company operates approximately 200 facilities worldwide, including manufacturing, sales, and R&D centers.

### The Target : DIAStudio & DIAScreen.

Delta’s DIAStudio Smart Machine Suite is an integrated engineering software package designed to simplify and reduce the time required for the setup process in machinery systems. Tasks such as product selection (Delta’s HMIs, PLCs, servo drives and motors, AC motor drives), PLC programming, quantitative parameter setting, machine tuning and HMI integration can all be executed smoothly on one platform with DIAStudio’s key tools

*   **DIAScreen:** Provides wizards for configuration, communication, Operation Interface design, Variable Tag sharing with DIADesigner and DIADesigner-AX

### How to do it ?

I’m not going to lie to you, in fact the process is the same as in this article (but there is a subtlety that means you should also finish reading this article before clicking on the link I’m giving you)

[Recover password from DELTA DOPSoft 4 .dpa
------------------------------------------

### Foreword

medium.com](https://medium.com/@biero-llagas/recover-password-from-delta-dopsoft-4-dpa-d65ee08823bc?source=post_page-----dbcd2289da19---------------------------------------)

There is always a problem with implementing the QcryptographicHash::addData(char const *, int) function, but this time on DIAscreen there is anti-debugging. But what is anti-debugging?

> Anti-debugging is a set of techniques used by software — common in malware and DRM-protected apps — to detect or hinder debuggers, preventing unauthorized analysis, reverse engineering, or tampering

So, as things stand at present, we cannot open Cheat Engine or x32 dbg. If you try to do anything active with these tools, the software will close itself.

I’m not going to give you a lecture on anti-debugging here. I think that if you’re reading this article, it’s because you want to know how to bypass security. So I’ll get straight to the point.

### ScyllaHide

[GitHub - x64dbg/ScyllaHide: Advanced usermode anti-anti-debugger. Forked from…
------------------------------------------------------------------------------

### Advanced usermode anti-anti-debugger. Forked from https://bitbucket.org/NtQuery/scyllahide - x64dbg/ScyllaHide

github.com](https://github.com/x64dbg/ScyllaHide?source=post_page-----dbcd2289da19---------------------------------------)

ScyllaHide is an advanced open-source x64/x86 user mode Anti-Anti-Debug library. It hooks various functions to hide debugging. This tool is intended to stay in user mode (ring 3). If you need kernel mode (ring 0) Anti-Anti-Debug, please see [TitanHide](https://github.com/mrexodia/titanhide). Forked from [NtQuery/ScyllaHide](https://bitbucket.org/NtQuery/scyllahide).

ScyllaHide supports various debuggers through plugins:

*   OllyDbg [v1](http://www.ollydbg.de) and [v2](http://www.ollydbg.de/version2.html)
*   [x64dbg](https://x64dbg.com)
*   [Hex-Rays IDA](https://www.hex-rays.com/products/ida/) v6 (not supported)
*   TitanEngine v2 ([original](http://www.reversinglabs.com/open-source/titanengine.html) and [updated](https://github.com/x64dbg/TitanEngine/) versions)

### How to add it to x32 dbg ?

If you have followed some of my tutorials, I have this expression, never reinvent the wheel, so I suggest you watch a video on installation.

After doing this, your x32dbg should not crash DIAScreen when you launch it.

Then just do the same actions as the DOPSoft article says, and you should be good.

![captionless image](https://miro.medium.com/v2/resize:fit:3832/format:webp/1*5Zabs9EEXjc7_rRDgXuFkg.png)

Conclusions
-----------

Just try not to hide problems that have been easy to find for at least six years behind anti-debugging.