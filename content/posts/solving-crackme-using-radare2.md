---
title: "Solving IOLI Crackmes Using Radare2"
date: 2020-07-13T00:08:32+05:30
---

As I was learning reverse engineering using r2, i tried the [IOLI crackmes]("https://github.com/Maijin/radare2-workshop-2015/tree/master/IOLI-crackme/bin-linux"). This post covers the solutions of these crackmes using r2.

## crackme0x00

Let's open the binary in radare 
`r2 -A crackme0x00`
This command will open crackme0x00 and analayse the binary (-A option is used for analyse) and give the r2 shell.

The information about the binary will be available in i command .
`iz -  prints strings in data sections`

As you can see , the password will be in the data section itself easy peasy !!
