---
title: "Debugging using radare2"
date: 2020-07-16T13:49:22+05:30
Description: "Debugging with r2"
Tags: [radare2,reverse,crackme]
Categories: [radare2,reverse,crackme]
DisableComments: false
---
 
Hi Y'ALL, before proceeding to solve other crackmes it is important to know how to debug in r2. This post is continuation of [Solving IOLI Crackmes Using Radare2 [Part-I]](post/solving-crackme-using-radare2).

Let's take the previous solved crackme0x02 and solve it by debugging it.

## Opening binary

We use `r2 -A crackme0x02` to open binary in r2 with analysing the binary on startup and can use other informational commands such as `i`,`a`,etc. To open binary in debugging mode, we have to use -d option.

`r2 -Ad crackme0x02`

The above one will open binary with analysing and starting in debugging mode.

```
root@n00b:~/Desktop/RE/radare2-workshop-2015/IOLI-crackme/bin-linux# r2 -Ad crackme0x03
Process with PID 14660 started...                                                                  
= attach 14660 14660                                                                               
bin.baddr 0x08048000                                                                               
Using 0x8048000                                                                                    
asm.bits 32                                                                                        
glibc.fc_offset = 0x00148                                                                          
[Invalid address from 0x080483ffith sym. and entry0 (aa)                                           
[x] Analyze all flags starting with sym. and entry0 (aa)                                           
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)                                         
[x] Check for objc references                                                                      
Warning: aao experimental on 32bit binaries                                                        
[x] Check for vtables                                                                              
[TOFIX: aaft can't run in debugger mode.ions (aaft)                                                
[x] Type matching analysis for all functions (aaft)                                                
[x] Propagate noreturn information                                                                 
[x] Use -AA or aaaa to perform additional experimental analysis.                                   
 -- The Hard ROP Cafe                                                                              
[0xf7fcb0b0]> 
```

## Reason for starting at higher address
0xf7fcb0b0 is the starting address the r2 has placed its debug. But lets see what this address is , is this a function or not

```
[0xf7fcb0b0]> afl
0x08048360    1 33           entry0                                                                
0x08048320    1 6            sym.imp.__libc_start_main                                             
0x080483b0    6 47           sym.__do_global_dtors_aux                                             
0x080483e0    4 50           sym.frame_dummy                                                       
0x080485a0    4 35           sym.__do_global_ctors_aux                                             
0x08048590    1 5            sym.__libc_csu_fini                                                   
0x080485c4    1 26           sym._fini                                                             
0x08048520    4 99           sym.__libc_csu_init                                                   
0x0804846e    4 42           sym.test                                                              
0x08048595    1 4            sym.__i686.get_pc_thunk.bx                                            
0x08048414    4 90           sym.shift                                                             
0x08048498    1 128          main                                                                  
0x08048350    1 6            sym.imp.printf                                                        
0x08048330    1 6            sym.imp.scanf                                                         
0x080482f8    1 23           sym._init                                                             
0x08048384    3 33           fcn.08048384                                                          
0x08048340    1 6            sym.imp.strlen                                                        
[0xf7fcb0b0]> 
```

oops , we see that all functions starts with address 0x08 and our starting seek is 0xf7 . ooooh, okay let's dig deeper regd this. So when we see the memory layout of c binaries , we can understand. 
![Memory Layout](/img/main/IOLI-r2/memlayout1.png)

As we see in the image, the binary text will be in low address and thats why the addresses of the functions starts with 0x08 whereis the stack address have higher address that is it starts with 0xf and in our case 0xf7fcb0b0 should be a stack address. But we all know that the stack addresses is read/write but how it can execute in stack . If we analyse the memory maps we can well understand 

```
[0xf7fcb0b0]> dm
0x08048000 - 0x08049000 - usr     4K s r-x /root/Desktop/RE/radare2-workshop-2015/IOLI-crackme/bin-linux/crackme0x03 /root/Desktop/RE/radare2-workshop-2015/IOLI-crackme/bin-linux/crackme0x03 ; map.root_Desktop_RE_radare2_workshop_2015_IOLI_crackme_bin_linux_crackme0x03.r_x                        
0x08049000 - 0x0804b000 - usr     8K s rw- /root/Desktop/RE/radare2-workshop-2015/IOLI-crackme/bin-linux/crackme0x03 /root/Desktop/RE/radare2-workshop-2015/IOLI-crackme/bin-linux/crackme0x03 ; map.root_Desktop_RE_radare2_workshop_2015_IOLI_crackme_bin_linux_crackme0x03.rw                         
0xf7fc6000 - 0xf7fc9000 - usr    12K s r-- [vvar] [vvar] ; map.vvar_.r                             
0xf7fc9000 - 0xf7fca000 - usr     4K s r-x [vdso] [vdso] ; map.vdso_.r_x                           
0xf7fca000 - 0xf7fcb000 - usr     4K s r-- /usr/lib32/ld-2.29.so /usr/lib32/ld-2.29.so             
0xf7fcb000 - 0xf7fe7000 * usr   112K s r-x /usr/lib32/ld-2.29.so /usr/lib32/ld-2.29.so ; map.usr_lib32_ld_2.29.so.r_x                                                                                 
0xf7fe7000 - 0xf7ff1000 - usr    40K s r-- /usr/lib32/ld-2.29.so /usr/lib32/ld-2.29.so ; map.usr_lib32_ld_2.29.so.r                                                                                   
0xf7ff2000 - 0xf7ff4000 - usr     8K s rw- /usr/lib32/ld-2.29.so /usr/lib32/ld-2.29.so ; map.usr_lib32_ld_2.29.so.rw
0xffdc6000 - 0xffde8000 - usr   136K s rw- [stack] [stack] ; map.stack_.rw
[0xf7fcb0b0]> 
``` 

The dm command will gives us the memory map regions allocated for the particular binary(crackme0x02) and it varies and it is allocated by the operating system for the processes. The * denotes the current execution region. The binary loaded in the current execution region is ld library which is loaded dynamically. Also permissions for the memory regions is also showed in 5th column and in our ld memory region it is r-x , so it has read and execute permission. Since there may be several entry functions for binaries , r2 will keep its first debugging in ld memory region and thats the reason we will start at higher address . Learn more about LD preload to understand more on why ld.so is loaded.

## debugging commands

It is important to know basic debugging commands in r2 

`d?` - debugging commands help


All debugging commands starts with `d`.

```
db - set breakpoint

dc - continue

ds - step into

dso - step over

dcu <function> - set breakpoint and continue which will hit the breakpoint at the function.
```

analysing memory commands starts with p

```
pfz @ <addr> - print value at addr as string(ends with 00)
pxw @ <addr> - print word from addr
px[b|w|q] <addr> -  print byte|word|quadword
p?,p??,p??? - help
```

## Visual Mode 

It will be difficult to debug without seeing cursor on the line executing by r2 and we can see that in visual mode.

```
v - open visual mode
p -  shift to different visual modes 
s - step into 
S - step over
q - quit visual mode
: - run r2 commands(you dont have to quit in order to run r2 commands)
```

It will look like this 
```
[0xf7fcb0b0 [xAdvc]0 0% 255 /root/Desktop/RE/radare2-workshop-2015/IOLI-crackme/bin-linux/crackme0x03]> pd $r @ loc._end+-268955504 # 0xf7fcb0b0                                                          
            ;-- eip:                                                                                                                                                                                      
            0xf7fcb0b0      89e0           mov eax, esp                                                                                                                                                   
            0xf7fcb0b2      83ec0c         sub esp, 0xc                                                                                                                                                   
            0xf7fcb0b5      50             push eax                                                                                                                                                       
            0xf7fcb0b6      e8e50c0000     call 0xf7fcbda0             ;[1]                                                                                                                               
            0xf7fcb0bb      83c410         add esp, 0x10                                                                                                                                                  
            0xf7fcb0be      89c7           mov edi, eax                                                                                                                                                   
            0xf7fcb0c0      e8dbffffff     call 0xf7fcb0a0             ;[2]                                                                                                                               
            0xf7fcb0c5      81c33b7f0200   add ebx, 0x27f3b                                                                                                                                               
            0xf7fcb0cb      8b8334f8ffff   mov eax, dword [ebx - 0x7cc]                                                                                                                                   
            0xf7fcb0d1      5a             pop edx                                                                                                                                                        
            0xf7fcb0d2      8d2484         lea esp, [esp + eax*4]                                                                                                                                         
            0xf7fcb0d5      29c2           sub edx, eax                                                                                                                                                   
            0xf7fcb0d7      52             push edx                                                                                                                                                       
            0xf7fcb0d8      8b8340000000   mov eax, dword [ebx + 0x40]                                                                                                                                    
            0xf7fcb0de      8d749408       lea esi, [esp + edx*4 + 8]                                                                                                                                     
            0xf7fcb0e2      8d4c2404       lea ecx, [esp + 4]                                                                                                                                             
            0xf7fcb0e6      89e5           mov ebp, esp                                                                                                                                                   
            0xf7fcb0e8      83e4f0         and esp, 0xfffffff0                                                                                                                                            
            0xf7fcb0eb      83ec0c         sub esp, 0xc                                                                                                                                                   
            0xf7fcb0ee      55             push ebp                                                                                                                                                       
            0xf7fcb0ef      56             push esi                                                                                                                                                       
            0xf7fcb0f0      51             push ecx                                                                                                                                                       
            0xf7fcb0f1      52             push edx                                                                                                                                                       
            0xf7fcb0f2      50             push eax                                                                                                                                                       
            0xf7fcb0f3      31ed           xor ebp, ebp                                                                                                                                                   
            0xf7fcb0f5      e876f00000     call 0xf7fda170             ;[3]           
```

## Panel Mode

My favorite is panel mode , i use panel mode for debugging

```
V - shift to panel mode
s - step into
S - step over
h,j,k,l - movement in panel mode
? - help 
w - window mode (shift to next window, other window operations)
e - create a new command in the panel
D - open disassembler
<spacebar> -  open graph view
```

