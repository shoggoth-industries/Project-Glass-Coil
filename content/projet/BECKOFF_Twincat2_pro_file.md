---
title: "Recover password from Beckoof TwinCat 2 (.pro) protected file."
description: "Cracking like it’s 2001: "
showDate: false
showReadingTime: false
---
Foreword
--------

The idea behind this blog post came to me after reading this article. I therefore recommend that you read it.

[Red Team - Compromising Critical Infrastructure by Reversing SCADA Software
---------------------------------------------------------------------------

### Discover how a red team compromises critical infrastructure by reversing SCADA software, escalating privileges in…

vrls.ws](https://vrls.ws/posts/2025/04/red-team-compromising-critical-infrastructure-by-reversing-scada-software/?source=post_page-----e3bd59561e7b---------------------------------------)

EWS software can be a good target , so here i am with a very old and easy one to bypass.

In the context ofgeneralist pentesting, software reverse engineering is often overlooked or quite rare because it takes a lot of time if you don’t have any specialization or at least some knowledge in this area.

The idea here is not to do things properly, but to get quick wins. The idea is to apply a very old method that still works in 2026, and it is generally one of the first techniques you learn, as well as a technique used in video game hacking.

### !!!!!!!!!! for educational purpose only !!!!!!!!!!!!!!!!!!!!!!!!!!!!!

Glossary
--------

*   ACL :[Access Contol List](https://fr.wikipedia.org/wiki/Access_Control_List)
*   EWS Software : Engineering Workstations (EWS) are specialized computers used by engineers for designing, configuring, monitoring, and maintaining complex systems, especially in industrial automation (SCADA/PLC environments)

Introductions
-------------

### Beckhoff

Beckhoff Automation is a German technology company specializing in PC-based control and automation, founded by Hans Beckhoff in 1980, evolving from his parents’ electrical business in Verl, Germany, and known for innovations like

[**EtherCAT**](https://www.google.com/search?q=EtherCAT&client=firefox-b-e&hs=FwVU&sca_esv=4ba3af865432350d&channel=entpr&sxsrf=ANbL-n6uvbFr7UgQln_JjWAhrVtq7Uc2jQ%3A1768648763964&ei=O3BraZHIOq6Fxc8PoeefkA8&ved=2ahUKEwiC1P-RupKSAxWZSfEDHXT9CxEQgK4QegQIARAB&uact=5&oq=beckoff+wikipedia+englais&gs_lp=Egxnd3Mtd2l6LXNlcnAaAhgCIhliZWNrb2ZmIHdpa2lwZWRpYSBlbmdsYWlzMgYQABgNGB4yBRAAGO8FMggQABiABBiiBDIFEAAY7wUyBRAAGO8FSOMMUPEBWIYMcAF4AZABAJgBzAagAbQQqgELMC40LjEuMS42LTG4AQPIAQD4AQGYAgigAuMQwgIWEAAYsAMY1gQYpgMYRxj4BRioAxiLA8ICChAAGLADGNYEGEfCAhwQLhiABBiwAxjRAxjSAxhDGMcBGKgDGIoFGIsDwgIGEAAYFhgemAMA4gMFEgExIECIBgGQBgmSBwsxLjQuMS4xLjYtMaAH8iOyBwswLjQuMS4xLjYtMbgH2RDCBwUwLjIuNsgHHIAIAA&sclient=gws-wiz-serp) and integrated industrial PCs, I/O, drives, and software (TwinCAT) for global manufacturing and process automation.

### The target : twincat 2 PLC Control.

Launched in 1996, TwinCat is a software suite that allows you to interface with and program PLCs.

To program it, you need the TwinCat 2 PLC Control software.

I’m not going to talk about how to program here.

When you create a project, you have the option of applying restrictions by group, which is a form of ACL proto.

Here, we are going to apply a password at Level 0.

![Applications of the password for user group.](https://miro.medium.com/v2/resize:fit:1394/format:webp/1*eFU5ZRfZtv0SLTMa4QHrPw.png)![Applications of ACL to specific sections of the file.](https://miro.medium.com/v2/resize:fit:1246/format:webp/1*ecvnydPlmZVbxhh7QfCVQg.png)

Once you have applied the password, save the file in .pro format, and you will need a password for it.

NOTE: The password that i have applied for my file is “13371337" (because leet is peak in the year 2000)

The original gangster : cheat engine.
-------------------------------------

Wikipedia say’s

> Cheat Engine (CE) is a memory scanner/debugger created by Eric Heijnen (“Byte, Dark”) for the Windows operating system in 2000. Cheat Engine is mostly used for cheating in computer games and is sometimes modified and recompiled to support new games. It searches for values input by the user with a wide variety of options that allow the user to find and sort through the computer’s memory. Cheat Engine can also create standalone trainers that can operate independently of Cheat Engine, often found on user forums or at the request of another user.

A tutorial and cheat engine integration that I highly recommend. It’s great for understanding the basics of the software.

![captionless image](https://miro.medium.com/v2/resize:fit:1022/format:webp/1*EsqYlB1RF-j27eSDVLbfzA.png)

Here a walktrought in case.

Methodologie
------------

The idea here is to start from the assumption that the password is not in plain text in the file (which is true) but that it is compared at some point to the password provided by the user in order to decode it.

The idea here will therefore be as follows

*   Check with Cheat Engine if the password is given and stored in memory.
*   See if actions access the latter in the context of a comparison with user input.
*   Try to modify the execution flow so that entering an incorrect password causes the file to be decrypted.

### 1: check password stored in memory

Ok so o have try to open the file with the password “696969" (again funny number whe are in the 2000)

We search by string and bam, we find both passwords (the one in the file and the one applied by the user

![captionless image](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*-rgi7-DLhDhgct1nWWNZKw.png)

### 2: check if something access this address

Then we will check if there are any elements that access the address.

![captionless image](https://miro.medium.com/v2/resize:fit:684/format:webp/1*HEqHfYmT101eP6eSZ-3k0A.png)

Then we restart the attempt to decrypt the file

Ok we have something.

![captionless image](https://miro.medium.com/v2/resize:fit:786/format:webp/1*aCajDQSTZ-AkT39LE6W9Tw.png)

Follow in assembly , and we are good.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Qcf_hC5lqInQZrBGFjqFiw.png)

Okay, so what do we have here?

*   jumps and conditional jumps (jmp, je)
*   pointers (a register surrounded by [] means that it is a pointer and that it will retrieve the contents of the address it points to)
*   and comparisons.

Great, that’s a lot. Place a few breakpoints to see the contents of the registers and restart the password attempt.

Our first breakpoint occurs at “mov al,[esi]”. Let’s try to follow the contents of esi to see what will be stored in al.

![captionless image](https://miro.medium.com/v2/resize:fit:692/format:webp/1*dKaGCQFyaQiqMARjj3FaKA.png)

And the incredible (really, it’s a bad CTF challenge) is that we can see the password applied by the user.

![captionless image](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*PPjXG13v7KP9ZA8I8txTtw.png)

Okay, so something must be happening in your brain. If [esi] is placed in al and contains the password that the user has given, and a few instructions later we compare al to ah (which will take the contents of [edi]), then the password could be in al, right?

Bingo, it’s 1996, don’t forget, back then it was really easy.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Pq9-Un_RKJRutycUWmpHPw.png)

Okay, great, but now we have the password, we’d still like to do some cracking. So let’s go back to the days when changing a jump could give you the contents of an industrial system file.

![captionless image](https://miro.medium.com/v2/resize:fit:450/format:webp/0*czr7ZVGasppAnhK3)

### 3: Try to modify the execution flow

For those, try modifying the confitinos function, which is the result of comparing the two passwords, I (meaning Jump equal), replacing it with jmp (meaning JUMp).

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*YzV4fuySG0l2jwAXRa4Yjg.png)

Remove the breack point , run and TADAAAAAA , you are a cracker

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*WVnHmLsMnX_CcPEczdk5LA.png)

NOTE: The jump you have modified appears to be used by other functions, so replace it to avoid freezing your TwinCat PLC Control.

![captionless image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*CcLaL7LpjFBWVqi4TEVwRg.png)

Conclusions
-----------

A short blog post to remind you that practicing off-track (outside of Hack the Box, Try Hack Me, and other platforms) is possible and even quite easy if you look for it a little.

It’s really an old-school technique on old-school software. Spoiler alert: this technique is still valid for software in Big 2026.

