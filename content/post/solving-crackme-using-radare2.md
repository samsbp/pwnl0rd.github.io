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

`iz-  prints strings in data sections`

As you can see , the password will be in the data section itself easy peasy !!.
![crackme0x00 img](/img/main/ioli-r2/crackme0x00_0.png)

You may wonder how the password is stored in data section, the point here is the global variables. In c, gloable variable's values are stored in data sections . So, what we can understand from this binary is that the password has been initialzed in a global variable and that's why we see the value of the global variable in our data section. Pls check out [this blog]("https://www.geeksforgeeks.org/memory-layout-of-c-program/")

## crackme0x01

Open the binary in radare using the same command as above

Now, if we use iz or izz (which gives all the strings in the binary whereas iz only gives strings in data section), we can't find our password for the binary. so let's analyse what the binary contains 

First we analyse the functions that are in the binary

`afl - gives the functions that are in the  binary`

![crackme0x01 img](/img/main/ioli-r2/crackme0x01_0.png)

As we can see from the above image, these functions are present in the binary. When the binary gets started the libc_csu_init finish is called which in-turns call the entry0 function which in-turns calls the main function. Main function is the function where the starting c logic for the binary will be present.

we disassemble only the main function using 

`pdf @ main`

where pdf is for print disassemble function. You can get more help on the command using ? ex: p?,p??,pd?? 

![crackme0x01 img](/img/main/ioli-r2/crackme0x01_1.png)



