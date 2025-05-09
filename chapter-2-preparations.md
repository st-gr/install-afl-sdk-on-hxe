---
title: Install AFL SDK on HANA Express - Chapter 2
author: st-gr
date: 5/6/2025
mainfont: Helvetica, Arial, sans-serif
fontsize: 18px
---

Install AFL SDK on HANA Express
===============================

***Guide to install the SAP Application Function Library SDK on SAP HANA Express***

**Author:** *st-gr*

[<< Previous Chapter](chapter-1-motivation.md) | [Content Table](README.md) | [Next Chapter >>](chapter-3-install-afl-sdk.md)

---

# Chapter Two: Preparations

This guide requires you to have the following software ready.

## HANA Express 2.0

You need to have HANA Express 2.0 installed and running on your virtualization software such as VMWare, VirtualBox, and others.
[SAP HANA trial][2] 

The standard HANA Express 2.0 *Open Virtual Appliance (.ova) container* can't be deployed on Hyper-V. If you want to use Hyper-V then you can follow my guide [SAP HANA Express 2.0 on Hyper-V][3].
I use the fully-loaded version with database + XS Advanced applications in this guide. The database only version works as well, but you won't have a WebIDE to play with.

## AFL SDK 2.0

You need to have an authorized user download software from SAP.
Since HANA Express has HDB AFL pre-installed you don't need to download `HDB_AFL_LINUX_X86_64`, but you need the HANA AFL SDK from SAP.

1. Navigate to [me.sap.com][4] and click the tile **Download Software**

2. Type in the search term `SAP HANA AFL SDK 2.0`

3. Find the `SAP HANA AFL SDK 2.0 (SUPPORT PACKAGES AND PATCHES)`

4. Choose the system architecture, for HANA Express 2.0, this is `LINUX ON X86_64 64BIT`.

5. Download the latest version of the **SAP HANA AFL Software Development Kit 2.0**
   This is currently `IMDBAFLSDK20_034_0-80002169.SAR`, release date November, 23 2018

![Download AFL SDK](/assets/me-sap-com-download-afl-sdk.png)

6. You need the SAPCAR tool to extract the archive contents, here run on Windows within a PowerShell 5.1:

````
PS> .\SAPCAR.EXE -xvf IMDBAFLSDK20_034_0-80002169.SAR
````
output:
````
SAPCAR: processing archive IMDBAFLSDK20_034_0-80002169.SAR (version 2.01)
x AFL_Developer_Manual_HANA2_SPS03.pdf
x AFL_SDK_Guide_HANA2_SPS03.pdf
x SAP_HANA_AFL_SDK_2_13_0.tgz
SAPCAR: 3 file(s) extracted
````

7. Read the manuals  
   Now it is time to read the SDK Guide and Developer Manual as I assume from now on that you have read those documents.
   
   If you've got scared (risk of destabilizing HDB, no support = the usual, etc.) of the risks involved then you should refrain from leveraging AFL altogether.
   
8. Read OSS notes

* 2190908 - Customer use of the Application Function Library SDK for SAP HANA
* 2046767 - Creating, dropping, and executing AFLLANG wrapper procedures

[2]: https://www.sap.com/products/data-cloud/hana/express-trial.html
[3]: https://github.com/st-gr/deploy-hxe-on-hyper-v/blob/main/README.md
[4]: https://me.sap.com/servicessupport

---

[<< Previous Chapter](chapter-1-motivation.md) | [Content Table](README.md) | [Next Chapter >>](chapter-3-install-afl-sdk.md)
