---
title: "Solving IOLI Crackme using Radare2 [Part-2]"
date: 2020-07-15T22:11:53+05:30
draft: true
---

Hi folks, this post is the continuation of [Solving IOLI Crackme using Radare2 Part-1](/post/solving-crackme-using-radare2/). In this blog, we will solve crackme0x02 and crackme0x03.

## crackme0x02

As there will no information about the password in data sections , we jump to analyse the assembly code.

The same as in [Part-1](/post/solving-crackme-using-radare2/) , we will open the binary using `r2 -A crackme0x02` and `pdf @ main`.

```
[0x08048330]> pdf @ main
            ; DATA XREF from entry0 @ 0x8048347
┌ 144: int main (int argc, char **argv, char **envp);
│           ; var uint32_t var_ch @ ebp-0xc
│           ; var signed int var_8h @ ebp-0x8
│           ; var int32_t input @ ebp-0x4
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
│           0x08048400      c70424488504.  mov dword [esp], str.IOLI_Crackme_Level_0x02 ; [0x8048548:4]=0x494c4f49 ; "IOLI Crackme Level 0x02\n" ; const char *format                                 
│           0x08048407      e810ffffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x0804840c      c70424618504.  mov dword [esp], str.Password: ; [0x8048561:4]=0x73736150 ; "Password: " ; const char *format                                                              
│           0x08048413      e804ffffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x08048418      8d45fc         lea eax, [input]
│           0x0804841b      89442404       mov dword [var_sp_4h], eax
│           0x0804841f      c704246c8504.  mov dword [esp], 0x804856c  ; [0x804856c:4]=0x50006425 ; const char *format                                                                                
│           0x08048426      e8e1feffff     call sym.imp.scanf          ; int scanf(const char *format)                                                                                                
│           0x0804842b      c745f85a0000.  mov dword [var_8h], 0x5a    ; 'Z' ; 90
│           0x08048432      c745f4ec0100.  mov dword [var_ch], 0x1ec   ; 492
│           0x08048439      8b55f4         mov edx, dword [var_ch]
│           0x0804843c      8d45f8         lea eax, [var_8h]
│           0x0804843f      0110           add dword [eax], edx
│           0x08048441      8b45f8         mov eax, dword [var_8h]
│           0x08048444      0faf45f8       imul eax, dword [var_8h]
│           0x08048448      8945f4         mov dword [var_ch], eax
│           0x0804844b      8b45fc         mov eax, dword [input]
│           0x0804844e      3b45f4         cmp eax, dword [var_ch]
│       ┌─< 0x08048451      750e           jne 0x8048461
│       │   0x08048453      c704246f8504.  mov dword [esp], str.Password_OK_: ; [0x804856f:4]=0x73736150 ; "Password OK :)\n" ; const char *format                                                    
│       │   0x0804845a      e8bdfeffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│      ┌──< 0x0804845f      eb0c           jmp 0x804846d
│      ││   ; CODE XREF from main @ 0x8048451
│      │└─> 0x08048461      c704247f8504.  mov dword [esp], str.Invalid_Password ; [0x804857f:4]=0x61766e49 ; "Invalid Password!\n" ; const char *format                                              
│      │    0x08048468      e8affeffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│      │    ; CODE XREF from main @ 0x804845f
│      └──> 0x0804846d      b800000000     mov eax, 0
│           0x08048472      c9             leave
└           0x08048473      c3             ret
[0x08048330]> 
```
Let's analyse the binary, 
The first 10 lines is for setting up the function's stack and after that IOLI_Crackme_Level_0x02 is loaded and sent as an argument to printf and it is called and the same for next printf. 

```
mat)                                                                                               
│           0x08048418      8d45fc         lea eax, [input]
│           0x0804841b      89442404       mov dword [var_sp_4h], eax
│           0x0804841f      c704246c8504.  mov dword [esp], 0x804856c  ; [0x804856c:4]=0x50006425 ; const char *format                                                                                
│           0x08048426      e8e1feffff     call sym.imp.scanf          ; int scanf(const char *format)
```

The above assembly is same as crackme0x01 and our input will be in var_4h and we will change the r2 variable name var_4h to input for better understanding as we saw in [Part-1](/post/solving-crackme-using-radare2/) using afvn.

```
│           0x0804842b      c745f85a0000.  mov dword [var_8h], 0x5a    ; 'Z' ; 90
│           0x08048432      c745f4ec0100.  mov dword [var_ch], 0x1ec   ; 492
│           0x08048439      8b55f4         mov edx, dword [var_ch]
│           0x0804843c      8d45f8         lea eax, [var_8h]
│           0x0804843f      0110           add dword [eax], edx
│           0x08048441      8b45f8         mov eax, dword [var_8h]
│           0x08048444      0faf45f8       imul eax, dword [var_8h]
│           0x08048448      8945f4         mov dword [var_ch], eax
│           0x0804844b      8b45fc         mov eax, dword [input]
│           0x0804844e      3b45f4         cmp eax, dword [var_ch]
│       ┌─< 0x08048451      750e           jne 0x8048461
│       │   0x08048453      c704246f8504.  mov dword [esp], str.Password_OK_: ; [0x804856f:4]=0x73736150 ; "Password OK :)\n" ; const char *format                                                    
│       │   0x0804845a      e8bdfeffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│      ┌──< 0x0804845f      eb0c           jmp 0x804846d
│      ││   ; CODE XREF from main @ 0x8048451
│      │└─> 0x08048461      c704247f8504.  mov dword [esp], str.Invalid_Password ; [0x804857f:4]=0x61766e49 ; "Invalid Password!\n" ; const char *format                                              
│      │    0x08048468      e8affeffff     call sym.imp.printf         ; int printf(const char *format)
```

This is the important part in the binary. The first mov statement will load 0x5a which is 90. But we see 'Z' in radare near the statement, because r2 tries to convert to char and 90 is 'Z' letter in ascii. So, we can deduce that there is a variable has value 90 and also another variable of value 492. Let's rename those variable names to int90 and int492. 

```
[0x08048330]> s main
[0x080483e4]> afvn int90 var_8h
[0x080483e4]> afvn int492 var_ch
[0x080483e4]> 
[0x080483e4]> pdf
            ; DATA XREF from entry0 @ 0x8048347                                                    
┌ 144: int main (int argc, char **argv, char **envp);                                              
│           ; var uint32_t int492 @ ebp-0xc                                                        
│           ; var signed int int90 @ ebp-0x8                                                       
│           ; var int32_t input @ ebp-0x4                                                         
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
│           0x08048400      c70424488504.  mov dword [esp], str.IOLI_Crackme_Level_0x02 ; [0x8048548:4]=0x494c4f49 ; "IOLI Crackme Level 0x02\n" ; const char *format                                 
│           0x08048407      e810ffffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x0804840c      c70424618504.  mov dword [esp], str.Password: ; [0x8048561:4]=0x73736150 ; "Password: " ; const char *format                                                              
│           0x08048413      e804ffffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x08048418      8d45fc         lea eax, [input]                                       
│           0x0804841b      89442404       mov dword [var_sp_4h], eax
│           0x0804841f      c704246c8504.  mov dword [esp], 0x804856c  ; [0x804856c:4]=0x50006425 ; const char *format                                                                                
│           0x08048426      e8e1feffff     call sym.imp.scanf          ; int scanf(const char *format)                                                                                                
│           0x0804842b      c745f85a0000.  mov dword [int90], 0x5a     ; 'Z' ; 90
│           0x08048432      c745f4ec0100.  mov dword [int492], 0x1ec   ; 492
│           0x08048439      8b55f4         mov edx, dword [int492]
│           0x0804843c      8d45f8         lea eax, [int90]
│           0x0804843f      0110           add dword [eax], edx
│           0x08048441      8b45f8         mov eax, dword [int90]
│           0x08048444      0faf45f8       imul eax, dword [int90]
│           0x08048448      8945f4         mov dword [int492], eax
│           0x0804844b      8b45fc         mov eax, dword [input]
│           0x0804844e      3b45f4         cmp eax, dword [int492]
│       ┌─< 0x08048451      750e           jne 0x8048461
│       │   0x08048453      c704246f8504.  mov dword [esp], str.Password_OK_: ; [0x804856f:4]=0x73736150 ; "Password OK :)\n" ; const char *format                                                    
│       │   0x0804845a      e8bdfeffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│      ┌──< 0x0804845f      eb0c           jmp 0x804846d
│      ││   ; CODE XREF from main @ 0x8048451
│      │└─> 0x08048461      c704247f8504.  mov dword [esp], str.Invalid_Password ; [0x804857f:4]=0x61766e49 ; "Invalid Password!\n" ; const char *format                                              
│      │    0x08048468      e8affeffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│      │    ; CODE XREF from main @ 0x804845f
│      └──> 0x0804846d      b800000000     mov eax, 0
│           0x08048472      c9             leave
└           0x08048473      c3             ret
[0x080483e4]> 
```
Ofcourse you may have doubt , why the variable int90 can't be a character. There are two reasons, one is the mov statement has dword[int90] which tells you to move 4 bytes (which is integer size) to int90 and if it was char then the statment should have `mov byte[int90],90` since char is 1 byte. The other reason is that as we analyse the code the binary does some addition and multipication operation on two variables if one is char and other is int then there should be a movzx kindof statement to convert int to char and as we dont find such we can surely say that those two variables are integers.

Continuing from where we left, two variables are initialized int90 and int492 . So the next statement will move int492 to edx and move int90 to eax and add those and the result will be in int492. The reason y there is lea instruction instead of mov statment is that if the assembly use mov statment for loading int90 then the added result will be stored in eax itself and there should be another line to move eax to int90, kindof inefficient and always compiler tries to give efficient compiled codes. The next statements will mutliply int90 and int90 and the result will be in int90 . So the operations that is done will be,

```
int90=int90+int492
int90=int90*int90

cmp input,int90
```
And the final value of the operation will be 338724.

## crackme0x03

As we know , we start to analyse the main function. The assembly code looks like this 

```
[0x08048360]> s main
[0x08048498]> afvn input var_4h
[0x08048498]> afvn int90 var_8h
[0x08048498]> afvn int492 var_ch
[0x08048498]> 
[0x08048498]> pdf @ main
            ; DATA XREF from entry0 @ 0x8048377
┌ 128: int main (int argc, char **argv, char **envp);
│           ; var int32_t int492 @ ebp-0xc
│           ; var signed int int90 @ ebp-0x8
│           ; var int32_t input @ ebp-0x4
│           ; var int32_t var_sp_4h @ esp+0x4
│           0x08048498      55             push ebp
│           0x08048499      89e5           mov ebp, esp
│           0x0804849b      83ec18         sub esp, 0x18
│           0x0804849e      83e4f0         and esp, 0xfffffff0
│           0x080484a1      b800000000     ov eax, 0
│           0x080484a6      83c00f         add eax, 0xf                ; 15
│           0x080484a9      83c00f         add eax, 0xf                ; 15
│           0x080484ac      c1e804         shr eax, 4
│           0x080484af      c1e004         shl eax, 4
│           0x080484b2      29c4           sub esp, eax
│           0x080484b4      c70424108604.  mov dword [esp], str.IOLI_Crackme_Level_0x03 ; [0x8048610:4]=0x494c4f49 ; "IOLI Crackme Level 0x03\n" ; const char *format                                 
│           0x080484bb      e890feffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x080484c0      c70424298604.  mov dword [esp], str.Password: ; [0x8048629:4]=0x73736150 ; "Password: " ; const char *format                                                              
│           0x080484c7      e884feffff     call sym.imp.printf         ; int printf(const char *format)                                                                                               
│           0x080484cc      8d45fc         lea eax, [input]
│           0x080484cf      89442404       mov dword [var_sp_4h], eax
│           0x080484d3      c70424348604.  mov dword [esp], 0x8048634  ; [0x8048634:4]=0x6425 ; const char *format                                                                                    
│           0x080484da      e851feffff     call sym.imp.scanf          ; int scanf(const char *format)                                                                                                
│           0x080484df      c745f85a0000.  mov dword [int90], 0x5a     ; 'Z' ; 90
│           0x080484e6      c745f4ec0100.  mov dword [int492], 0x1ec   ; 492
│           0x080484ed      8b55f4         mov edx, dword [int492]
│           0x080484f0      8d45f8         lea eax, [int90]
│           0x080484f3      0110           add dword [eax], edx
│           0x080484f5      8b45f8         mov eax, dword [int90]
│           0x080484f8      0faf45f8       imul eax, dword [int90]
│           0x080484fc      8945f4         mov dword [int492], eax
│           0x080484ff      8b45f4         mov eax, dword [int492]
│           0x08048502      89442404       mov dword [var_sp_4h], eax
│           0x08048506      8b45fc         mov eax, dword [input]
│           0x08048509      890424         mov dword [esp], eax
│           0x0804850c      e85dffffff     call sym.test
│           0x08048511      b800000000     mov eax, 0
│           0x08048516      c9             leave
└           0x08048517      c3             ret
[0x08048498]> 
m
```
If you have gone through crackme0x02 , you can understand the assembly till the call sym.test. 

```
│           0x080484ff      8b45f4         mov eax, dword [int492]
│           0x08048502      89442404       mov dword [var_sp_4h], eax
│           0x08048506      8b45fc         mov eax, dword [input]
│           0x08048509      890424         mov dword [esp], eax
│           0x0804850c      e85dffffff     call sym.test

```

The next instructions after int492 has value 338724. This value  and our input is sent to sym.test as an argument when called.

