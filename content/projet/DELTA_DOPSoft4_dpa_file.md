---
title: "Recover password from DELTA DOPSoft 4 .dpa protected file"
description: "Patch password verifications from the software."
showDate: false
showReadingTime: false
author: biero-llagas
---


Recover password from DELTA DOPSoft 4 .dpa protected file and patch password verifications from the software.
=============================================================================================================


Foreword
--------

### !!!!!!!!!! for educational purpose only !!!!!!!!!!!!!!!!!!!!!!!!!!!!!

This article is part of a series of articles focused on reverse engineering EWS software to determine the security of project files and potentially bypass the security measures applied to them.

The series includes the following items in addition to this one.

*   [https://medium.com/@biero-llagas/cracking-like-its-2001-recover-password-from-beckoof-twincat-2-pro-protected-file-e3bd59561e7b](https://medium.com/@biero-llagas/cracking-like-its-2001-recover-password-from-beckoof-twincat-2-pro-protected-file-e3bd59561e7b)

NOTE 1: For the time being, the actions will remain in Proof of Concept format. So if you are here to get a direct patch without getting your hands dirty, this article is not for you.

NOTE 2: The goal here is not to completely reverse engineer the application, but just to achieve the objectives (password recovery and application patching), so it won’t really be reverse engineering or theft if there is a simple solution.

Glossary
--------

*   ACL :[Access Contol List](https://fr.wikipedia.org/wiki/Access_Control_List)
*   EWS Software : Engineering Workstations (EWS) are specialized computers used by engineers for designing, configuring, monitoring, and maintaining complex systems, especially in industrial automation (SCADA/PLC environments)

Tools
-----

*   [cheat engine](https://cheatengine.org/)
*   [X32 dbg](https://x64dbg.com/)

Introductions
-------------

### Delta

Delta Electronics, Inc. (Chinese: 台達電子; also known as DELTA or Delta Electronics) is a Taiwanese electronics manufacturing company. Its headquarters are in Neihu, Taipei. It is known for its DC industrial and computer fans, data center rectifiers and switching power supplies. The company operates approximately 200 facilities worldwide, including manufacturing, sales, and R&D centers.

### The Target : DOPSoft V4

A word form DELTA EMEA

> DOPSoft V4 is engineering software used to design, configure, and program Delta Electronics HMI (Human Machine Interface) panels, specifically the DOP-100 series HMIs. Its main purpose is to create graphical operator screens and handle communication between the HMI and PLCs/other automation devices in industrial control systems

![captionless image](https://miro.medium.com/v2/resize:fit:522/format:webp/0*fNHTjUsd4vrpI5eu)

The DOPSoft (V4) software have reached its end of life (EOL) in May 2023, which in the world of industry is not that old (a little less than three years since this article was written).

The cool thing is that the software is available for free at the installation site, so this time you can repeat the steps I show you.

[Download Center
---------------

### Edit description

downloadcenter.deltaww.com](https://downloadcenter.deltaww.com/en-US/DownloadCenter?v=1&q=DIAScreen&sort_expr=cdate&sort_dir=DESC&source=post_page-----d65ee08823bc---------------------------------------)

One thing should catch your attention when you arrive at the download page.

> “might be concerned with cyber security”

![captionless image](https://miro.medium.com/v2/resize:fit:1332/format:webp/1*zeMnS48Atszh6rUal5Za7g.png)

Here more context in their EOL term

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*lOLgkeJY7sUyav9DJcaRXg.png)

Buffer overlfow that cool , but that is not what we are interested in today.

Install it , and normaly you will see this screen.

![captionless image](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*0aqHlpOGJaHInf-Yr-_r-w.png)

NOTE: like for the other article in this series i will not go in the details of the use of the interface , the only stuff that interest us is the management of the protections of the prohect file.

Create the protected file
-------------------------

Create a random project for any GUI (the GUI version does not seem to have any impact on the management of project file security).

![captionless image](https://miro.medium.com/v2/resize:fit:752/format:webp/1*68n8T7GrPO2JJzIB6nlCKA.png)

Once the project has been created, password protection must be enabled.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ZtBWvNBtC6fN61MwtqQfdg.png)

Let’s be a little playful and activate maximum security.

NOTE: like the Beckoof TwinCat 2 PLC Controle ther is a form of ACL in the project , you can apply password on defined group , and give rifht to those group in your project.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*M6RSZkjkHtHBucvppCBfRA.png)

Save the project and try to reopen it.

![captionless image](https://miro.medium.com/v2/resize:fit:752/format:webp/1*KA8qWg0onQiXfFoQXylRUw.png)

You will be asked for the password

Let’s dig deeper (Patch the diff and open any password protected file)
----------------------------------------------------------------------

NOTE: the next sections will be on how to retrieve the password of the file , if it’s sometions that you are interested in , go directly to this sections.

For the first step, we’ll start with Cheat Engine to see if we can find any clues about the password execution chain.

NOTE: The example password for this article will be “POTTER.”

Launch Cheat Engine, enter “potter” in the DOPSoft password field, and search for “POTTER” as a character string.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*jZpWI7y0jGdest9U07CiLg.png)

That’s a good start.

Next, we’ll see if there are any actions taking place (right-click on the POTTER value in the step variable, then select “Find or what accesses this address”).

![captionless image](https://miro.medium.com/v2/resize:fit:754/format:webp/1*U1SZuOpmmhHbcitu-dmatQ.png)

Okay that something at least.

![captionless image](https://miro.medium.com/v2/resize:fit:970/format:webp/1*F5_VwbxhS9eiINTkFoCNAg.png)

Let’s take a look at the first one and examine the assembler view. The first thing we can see is the function calls from the Qt5Core DLL.

The application is written with Qt Creator. I am not an expert in this framework, so I may say something silly about the use of some of these classes and functions.

![captionless image](https://miro.medium.com/v2/resize:fit:566/format:webp/1*9YA6bH2kFhDCTyM21NmAHw.png)

avencon un peut and there we can see a function that might interest us (Qt5Core:QCryptogrphicHash::hash)

![captionless image](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*p5ysQMuNNRJjwpCQpHGJAw.png)

According to the Qt Creator documentation, this class is used to generate hashes.

[https://doc.qt.io/archives/qt-5.15/qcryptographichash.html](https://doc.qt.io/archives/qt-5.15/qcryptographichash.html)

And if we go a little further (in the executions flow), we can see that there are indeed two hashes in memory

![captionless image](https://miro.medium.com/v2/resize:fit:686/format:webp/1*ZReJp4Tn7MC6d7E1XFSttg.png)

With practice, you can see that it has the form of sha256 (you can confirm this by the parameters applied to Qt5Core.qcryptographichash) or just by checking the password applied to confirm that this is indeed the case.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*PXn_PxGgT__5ghhYte-dIQ.png)

Indeed, the hash corroborates this.

So we can see that our password will be hashed. And potentially compared with the password in the project file. Let’s try to confirm this hypothesis.

With a little research, we can find this comparison.

Here, both hashes are in the stack, and they will be loaded into EAX (project file hash) and ECX (user input hash) respectively.

![captionless image](https://miro.medium.com/v2/resize:fit:2514/format:webp/1*DdFHRJwzw2LoW5wY0jXLPQ.png)

So if we patch for this comparison, and apply the recovery in the stack, the real password, and patch, we should be able to open it without any problems, right?

![captionless image](https://miro.medium.com/v2/resize:fit:2514/format:webp/1*x9YNYip5DK7qa-94bthuEg.png)

L’etsgo that work

![captionless image](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*NuSun5vu53zo9StPSMDrwg.png)

NOTE: I haven’t extensively tested whether this patch could cause problems with TwinCat 2, but it seems to work without too many issues.

### Clear text password before hashing.

Indeed, one of the objectives was to bypass security and also to recover the password for the project file.

For this one, I’ll spare you the messy screenshots, and we’ll get straight to the point.

The class [https://doc.qt.io/archives/qt-5.15/qcryptographichash.html](https://doc.qt.io/archives/qt-5.15/qcryptographichash.html) contains functions, one of which may be of interest to us: addData.

![captionless image](https://miro.medium.com/v2/resize:fit:1142/format:webp/1*TvU55pVoMVxx5hwNSl-CKw.png)

This function, if I understand correctly, will apply an entry to the hashing function, and therefore apply a string to be hashed.

If we can place ourselves on the call to this function, we may be able to find the password for the file that will be hashed.

In x32 dbg, you can set breakpoints at certain locations. Try it in qt5core.dll, the function QcryptographicHash::addData(char const *, int).

![captionless image](https://miro.medium.com/v2/resize:fit:3322/format:webp/1*hfcrVHY0E-gRZxM6B2c_8w.png)

We apply the breakpoint and launch the password comparison function as in part 1, and boom, we get the password applied by the user first.

![captionless image](https://miro.medium.com/v2/resize:fit:3240/format:webp/1*puHe5I9rzBJoxFM51Sglbw.png)

And the file password.

![captionless image](https://miro.medium.com/v2/resize:fit:3240/format:webp/1*IBUWBoW6mLTrHQ-wMwB1ng.png)

Wow that was at least a entry level CTF challenge.

![captionless image](https://miro.medium.com/v2/resize:fit:450/format:webp/0*dZ1M3f4LFYXZl5eV)

Conclusions
-----------

Crypto implementations are quite difficult to do, and even more so when the system is self-contained and the entire process of checking/comparing is done directly in the file.

I therefore advise you to follow Delta’s advice and migrate to the latest application and avoid using this type of software. We have just discovered that there is in fact no security applied, just a layer of obfuscation.