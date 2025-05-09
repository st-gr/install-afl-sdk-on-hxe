---
title: Install AFL SDK on HANA Express - Chapter 1
author: st-gr
date: 5/6/2025
mainfont: Helvetica, Arial, sans-serif
fontsize: 18px
---

Install AFL SDK on HANA Express
===============================

***Guide to install the SAP Application Function Library SDK on SAP HANA Express***

**Author:** *st-gr*

[Content Table](README.md) | [Next Chapter >>](chapter-2-preparations.md)

---

# Chapter One: Motivation
Creating AFL plugins enables you to enhance SAP HANA functionality beyond the standard deployed functional scope. For example, there is ***no native Base64 encoding and decoding*** available on HANA, and SAP is currently not considering this for the near future, as seen in [Provide Base64 decode/encode as native database function][1]. With Cloud and SaaS deployments, vendors like to lock you into their walled garden, making extensibility even more important. Having such low-level extensibility capabilities is detrimental to the goal of having a stable cloud environment where the same service/server is shared among many customers. However, this should not apply when one operates dedicated instances such as S/4 HANA private cloud.

Anything that can be compiled in C/C++ within the AFL constraints can be implemented. Think big!

[1]: https://influence.sap.com/sap/ino/#/idea/295931

---

[Content Table](README.md) | [Next Chapter >>](chapter-2-preparations.md)
