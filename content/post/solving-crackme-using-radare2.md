---
title: "Solving IOLI Crackmes Using Radare2 [Part-I]"
date: 2020-07-13T00:08:32+05:30
---

As I was learning reverse engineering using r2, i tried the [IOLI crackmes](https://github.com/Maijin/radare2-workshop-2015/tree/master/IOLI-crackme/bin-linux). This post covers the solutions of two crackmes(0x00,0x01) using r2.

## crackme0x00

Let's open the binary in radare

`r2 -A crackme0x00`

This command will open crackme0x00 and analayse the binary (-A option is used for analyse) and give the r2 shell.

The information about the binary will be available in i command .

`iz-  prints strings in data sections`

As you can see , the password will be in the data section itself easy peasy !!.
```
root@n00b:~/Desktop/RE/radare2-workshop-2015/IOLI-crackme/bin-linux# r2 -A crackme0x00
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Check for objc references
Warning: aao experimental on 32bit binaries
[x] Check for vtables
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information
[x] Use -AA or aaaa to perform additional experimental analysis.
 -- There's more than one way to skin a cat
[0x08048360]> iz
[Strings]
nth paddr      vaddr      len size section type  string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00000568 0x08048568 24  25   .rodata ascii IOLI Crackme Level 0x00\n
1   0x00000581 0x0848581 10  11   .rodata ascii Password: 
2   0x0000058f 0x0804858f 6   7    .rodata ascii 250382
3   0x00000596 0x08048596 18  19   .rodata ascii Invalid Password!\n
4   0x000005a9 0x080485a9 15  16   .rodata ascii Password OK :)\n

[0x08048360]> 
```
You may wonder how the password is stored in data section, the point here is the global variables. In c, gloable variable's values are stored in data sections . So, what we can understand from this binary is that the password has been initialzed in a global variable and that's why we see the value of the global variable in our data section. Pls check out [this blog](https://www.geeksforgeeks.org/memory-layout-of-c-program/)

## crackme0x01

Open the binary in radare using the same command as above

Now, if we use iz or izz (which gives all the strings in the binary whereas iz only gives strings in data section), we can't find our password for the binary. so let's analyse what the binary contains 

First we analyse the functions that are in the binary

`afl - gives the functions that are in the  binary`

```
[0x08048330]> afl
0x08048330    1 33           entry0
0x080482fc    1 6            sym.imp.__libc_start_main
0x08048380    6 47           sym.__do_global_dtors_aux
0x080483b0    4 50           sym.frame_dummy
0x080484e0    4 35           sym.__do_global_ctors_aux
0x080484d0    1 5            sym.__libc_csu_fini
0x08048504    1 26           sym._fini
0x08048460    4 99           sym.__libc_csu_init
0x080484d5    1 4            sym.__i686.get_pc_thunk.bx
0x080483e4    4 113          main
0x080482d4    1 23           sym._init
0x08048354    3 33           fcn.08048354
0x0804830c    1 6            sym.imp.scanf
0x0804831c    1 6            sym.imp.printf
[0x08048330]> 
```
As we can see from the above, these functions are present in the binary. When the binary gets started the libc_csu_init is called which in-turns call the entry0 function which in-turns calls the main function. Main function is the function where the starting c logic for the binary will be present.

we disassemble only the main function using 

`pdf @ main`

where pdf is for print disassemble function. You can get more help on the command using ? ex: p?,p??,pd?? 

```
[0x080483e4]> pdf @ main
            ; DATA XREF from entry0 @ 0x8048347                                                    
┌ 113: int main (int argc, char **argv, char **envp);                                              
│           ; var uint32_t var_4h @ ebp-0x4                                                        
│           ; var int32_t var_sp_4h @ esp+0x4
│           0x080483e4      55             push ebp
│           0x080483e5      89e5           mov ebp, esp
│           0x080483e7      83ec18         sub esp, 0x18
│           0x080483ea      83e4f0         and esp, 0xfffffff0
│           0x080483ed      b800000000     mov eax, 0
│           0x080483f2      83c00f         add eax, 0xf                ; 15
│           0x080483f5      83c00f         add eax, 0xf                ; 15
│           0x080483f8      c1e804         shr eax, 4
│           0x080483fb      c1e004         shl eax, 4
│           0x080483fe      29c4           sub esp, eax
│           0x08048400      c70424288504.  mov dword [esp], str.IOLI_Crackme_Level_0x01 ; [0x8048528:4]=0x494c4f49 ; "IOLI Crackme Level 0x01\n" ; const char *format                                 
│           0x08048407      e810ffffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x0804840c      c70424418504.  mov dword [esp], str.Password: ; [0x8048541:4]=0x73736150 ; "Password: " ; const char *format                                                              
│           0x08048413      e804ffffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x08048418      8d45fc         lea eax, [var_4h]
│           0x0804841b      89442404       mov dword [var_sp_4h], eax
│           0x0804841f      c704244c8504.  mov dword [esp], 0x804854c  ; [0x804854c:4]=0x49006425 ; const char *format                                                                                
│           0x08048426      e8e1feffff     call sym.imp.scanf          ; int scanf(const char *format)                                                                                                
│           0x0804842b      817dfc9a1400.  cmp dword [var_4h], 0x149a
│       ┌─< 0x08048432      740e           je 0x8048442
│       │   0x08048434      c704244f8504.  mov dword [esp], str.Invalid_Password ; [0x804854f:4]=0x61766e49 ; "Invalid Password!\n" ; const char *format                                              
│       │   0x0804843b      e8dcfeffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│      ┌──< 0x08048440      eb0c           jmp 0x804844e
│      ││   ; CODE XREF from main @ 0x8048432
│      │└─> 0x08048442      c70424628504.  mov dword [esp], str.Password_OK_: ; [0x8048562:4]=0x73736150 ; "Password OK :)\n" ; const char *format                                                    
│      │    0x08048449      e8cefeffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│      │    ; CODE XREF from main @ 0x8048440
│      └──> 0x0804844e      b800000000     mov eax, 0
│           0x08048453      c9             leave
└           0x08048454      c3             ret
[0x080483e4]> 

```

Before going straight to understand the assembly code , read the [intel blog](https://software.intel.com/content/www/us/en/develop/articles/introduction-to-x64-assembly.html) and also [this helper for the intel blog](https://github.com/samsbp/learnReverse/blob/master/week1/IntelRead.md)

As you have guessed , the binary is a 32 bit binary . There are two ways to find it, one is to use file command other we can guess by looking at the assembly itself since eax,edx and all e's in the registers denotes 32 bit registers if the eax was rax and edx as rdx then it is a 64 bit registers which in-turn denotes 64 bit binary.

The first 10 line of assembly code is for function prologue which is used to setup the stack . Y stack? because the main function is called and when a function is called a stack will be created to contain the local varibles of the called functions. Note: stack starts from higher addresses (read on memory layout in c for more)

After the function stack is set, printf function is called 

So we can start to understand the assembly from the 11th line.

```
│           0x08048400      c70424288504.  mov dword [esp], str.IOLI_Crackme_Level_0x01 ; [0x8048528:4]=0x494c4f49 ; "IOLI Crackme Level 0x01\n" ; const char *format                                 
│           0x08048407      e810ffffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x0804840c      c70424418504.  mov dword [esp], str.Password: ; [0x8048541:4]=0x73736150 ; "Password: " ; const char *format                                                              
│           0x08048413      e804ffffff     call sym.imp.printf
```

The mov command will move the IOLI_Crackme_Level_0x01 string to esp's location so esp is nothing but the stack pointer and we can deduce that the string is passed as an argument to printf function and the print will be called and the string IOLI_Crackme_Level_0x01 will be displayed and the again the next printf will be called in the same manner.

```
│           0x08048418      8d45fc         lea eax, [var_4h]
│           0x0804841b      89442404       mov dword [var_sp_4h], eax
│           0x0804841f      c704244c8504.  mov dword [esp], 0x804854c  ; [0x804854c:4]=0x49006425 ; const char *format                                                                                
│           0x08048426      e8e1feffff     call sym.imp.scanf          ; int scanf(const char *format)                                                                                                
│           0x0804842b      817dfc9a1400.  cmp dword [var_4h], 0x149a
```

The next command is lea which will load the address of var_4h , so var_4h is the location where our input is stored. How do we know that it is in var_4h the input will be stored , it is because the scanf contains two arguments , one is the format string and the other is the variable to store the input. In assembly the argument will be loaded from right to left( it depends on [calling conventions](https://en.wikipedia.org/wiki/X86_calling_conventions) ) and since the var_4h is loaded first, the input will be in var_4h. As you can see the format string is the next one to be loaded which is available in 0x804854c. We can extract the string from 0x804854c to see the stored format string by using pfz

`pfz @ 0x804854c` 

which is print format as string @ addr
```
[0x08048330]> pfz @ 0x804854c
0x0804854c = "%d"
```

The input will be stored in var_4h so lets rename the var_4h as input for better understanding , we can rename those variable names which is given by r2 (note: r2 is the one which provides these varaible names for our understanding and it will not be in the binary) using afvn

`afvn input var_4h`

use afv? for understanding what the command does 

But before issuing the afvn command we should [seek](https://radare.gitbooks.io/radare2book/basic_commands/seeking.html) the address to main function so that r2 can find tha var_4h which is only available in main function only ,so it will look like this 

```
[0x08048330]> s main
[0x080483e4]> afvn input var_4h
[0x080483e4]> pdf @ main
            ; DATA XREF from entry0 @ 0x8048347
┌ 113: int main (int argc, char **argv, char **envp);
│           ; var uint32_t input @ ebp-0x4
│           ; var int32_t var_sp_4h @ esp+0x4
│           0x080483e4      55             push ebp
│           0x080483e5      89e5           mov ebp, esp
│           0x080483e7      83ec18         sub esp, 0x18
│           0x080483ea      83e4f0         and esp, 0xfffffff0
│           0x080483ed      b800000000     mov eax, 0
│           0x080483f2      83c00f         add eax, 0xf                ; 15
│           0x080483f5      83c00f         add eax, 0xf                ; 15
│           0x080483f8      c1e804         shr eax, 4
│           0x080483fb      c1e004         shl eax, 4
│           0x080483fe      29c4           sub esp, eax
│           0x08048400      c70424288504.  mov dword [esp], str.IOLI_Crackme_Level_0x01 ; [0x8048528:4]=0x494c4f49 ; "IOLI Crackme Level 0x01\n" ; const char *format                                 
│           0x08048407      e810ffffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x0804840c      c70424418504.  mov dword [esp], str.Password: ; [0x8048541:4]=0x73736150 ; "Password: " ; const char *format                                                              
│           0x08048413      e804ffffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x08048418      8d45fc         lea eax, [input]
│           0x0804841b      89442404       mov dword [var_sp_4h], eax
│           0x0804841f      c704244c8504.  mov dword [esp], 0x804854c  ; [0x804854c:4]=0x49006425 ; const char *format                                                                                
│           0x08048426      e8e1feffff     call sym.imp.scanf          ; int scanf(const char *format)                                                                                                
│           0x0804842b      817dfc9a1400.  cmp dword [input], 0x149a
│       ┌─< 0x08048432      740e           je 0x8048442
│       │   0x08048434      c704244f8504.  mov dword [esp], str.Invalid_Password ; [0x804854f:4]=0x61766e49 ; "Invalid Password!\n" ; const char *format                                              
│       │   0x0804843b      e8dcfeffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│      ┌──< 0x08048440      eb0c           jmp 0x804844e
│      ││   ; CODE XREF from main @ 0x8048432
│      │└─> 0x08048442      c70424628504.  mov dword [esp], str.Password_OK_: ; [0x8048562:4]=0x73736150 ; "Password OK :)\n" ; const char *format                                                    
│      │    0x08048449      e8cefeffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│      │    ; CODE XREF from main @ 0x8048440
│      └──> 0x0804844e      b800000000     mov eax, 0
│           0x08048453      c9             leave
└           0x08048454      c3             ret
[0x080483e4]> 
```

The last part of the binary is checking the password input value to the correct value using cmp

```
│           0x0804842b      817dfc9a1400.  cmp dword [input], 0x149a
│       ┌─< 0x08048432      740e           je 0x8048442
│       │   0x08048434      c704244f8504.  mov dword [esp], str.Invalid_Password ; [0x804854f:4]=0x61766e49 ; "Invalid Password!\n" ; const char *format                                              
│       │   0x0804843b      e8dcfeffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│      ┌──< 0x08048440      eb0c           jmp 0x804844e
│      ││   ; CODE XREF from main @ 0x8048432
│      │└─> 0x08048442      c70424628504.  mov dword [esp], str.Password_OK_: ; [0x8048562:4]=0x73736150 ; "Password OK :)\n" ; const char *format                                                    
│      │    0x08048449      e8cefeffff     call sym.imp.printf         ; int printf(const char *format)
```

As said, the cmp instruction compares our input variable which contains our input for password to 0x149. so 0x149 is in hex ofcouse you can use python or other ways to convert hex to int but in radare2 we can convert using `?` 

```
[0x080483e4]> ? 0x149a
int32   5274
uint32  5274
hex     0x149a
octal   012232
unit    5.2K
segment 0000:049a
string  "\x9a\x14"
fvalue: 5274.0
float:  0.000000f
double: 0.000000
binary  0b0001010010011010
trits   0t21020100
[0x080483e4]> 
```

As you can see, the int format for our hex value 0x149a is 5274. And if we continue the assembly code, after cmp it jumps if equal (je) to 0x8048442 which is nothing  but printing password ok which means we found the password ;). 

I know it is not good to continue after this as you may feel brainfucked, it's okay . Start solving the crackme's and if you can't solve it, you can refer this blog.

Will see you in the next blog post [Part-2].
