---
title: Deploy HANA Express 2.0 on Microsoft Hyper-V - Chapter 3
author: st-gr
date: 5/1/2025
mainfont: Helvetica, Arial, sans-serif
fontsize: 18px
---

SAP HANA Express 2.0 on Hyper-V
===============================

***Guide to deploy HANA Express 2.0 on Microsoft Hyper-V***

**Author:** *st-gr*

[<< Previous Chapter](chapter-2-preparations.md) | [Content Table](README.md) | [Next Chapter >>](chapter-4-create-vm.md)

---

# Chapter Three: Convert the virtual appliance

Your `HXE` download folder should look similar to this:

![Folder containing the downloaded files - note that I also downloaded `qemu-img` there](/assets/hxe-downloads-folder.png)

Depending on your machines I/O performance you will have the converted VHDX file after about 5 minutes time. It grew in size and is now 62 GB proofing that VHDX images are less efficient compared to VMDK.

![Converted VHDX hard disk image](/assets/qemu-img-convert-to-vhdx.png)

[8]: https://en.wikipedia.org/wiki/VHD_(file_format)
[9]: https://en.wikipedia.org/wiki/Fabrice_Bellard

---

[<< Previous Chapter](chapter-2-preparations.md) | [Content Table](README.md) | [Next Chapter >>](chapter-4-create-vm.md)
