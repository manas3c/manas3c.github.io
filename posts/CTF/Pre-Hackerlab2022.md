---
layout: default
title : Redteam-TG -HackerLab2022 challanges writeups
---

## Ecowas Portal chall

```
Is my program secure?
Flag: CTF_*
```
We’re given a binary, let's run it
```
┌──(kali㉿kali)-[~/CTFs/HackerlabCTF/Basic/Ecowas]
└─$ ./ecowas_portal                           
ECOWAS ADMIN PORTAL : 
test
Flag wrong. Try again.  
```
Strings won’t help us.
I like using gdb for static analysis. Let’s look at main.

```
┌──(kali㉿kali)-[~/CTFs/HackerlabCTF/Basic/Ecowas]
└─$ gdb -q ecowas_portal
Reading symbols from ecowas_portal...
(No debugging symbols found in ecowas_portal)
(gdb) run
Starting program: /home/kali/CTFs/HackerlabCTF/Basic/Ecowas/ecowas_portal 
ECOWAS ADMIN PORTAL : 
test
Flag wrong. Try again.[Inferior 1 (process 19161) exited with code 01]
(gdb) disass main
Dump of assembler code for function main:
   0x000055555555517e <+0>:     push   rbp
   0x000055555555517f <+1>:     mov    rbp,rsp
   0x0000555555555182 <+4>:     push   rbx
   0x0000555555555183 <+5>:     sub    rsp,0x118
   0x000055555555518a <+12>:    lea    rax,[rip+0xe73]        # 0x555555556004
   0x0000555555555191 <+19>:    mov    rdi,rax
   0x0000555555555194 <+22>:    call   0x555555555030 <puts@plt>
   0x0000555555555199 <+27>:    mov    DWORD PTR [rbp-0x120],0x2f
   0x00005555555551a3 <+37>:    mov    DWORD PTR [rbp-0x11c],0x41
   0x00005555555551ad <+47>:    mov    DWORD PTR [rbp-0x118],0x30
   0x00005555555551b7 <+57>:    mov    DWORD PTR [rbp-0x114],0x48
   0x00005555555551c1 <+67>:    mov    DWORD PTR [rbp-0x110],0x4a
   0x00005555555551cb <+77>:    mov    DWORD PTR [rbp-0x10c],0x25
   0x00005555555551d5 <+87>:    mov    DWORD PTR [rbp-0x108],0x27
   0x00005555555551df <+97>:    mov    DWORD PTR [rbp-0x104],0x1a
   0x00005555555551e9 <+107>:   mov    DWORD PTR [rbp-0x100],0x27
   0x00005555555551f3 <+117>:   mov    DWORD PTR [rbp-0xfc],0x57
   0x00005555555551fd <+127>:   mov    DWORD PTR [rbp-0xf8],0x15
   0x0000555555555207 <+137>:   mov    DWORD PTR [rbp-0xf4],0x49
   0x0000555555555211 <+147>:   mov    DWORD PTR [rbp-0xf0],0x10
   0x000055555555521b <+157>:   mov    DWORD PTR [rbp-0xec],0x2d
   0x0000555555555225 <+167>:   mov    DWORD PTR [rbp-0xe8],0x11
   0x000055555555522f <+177>:   mov    DWORD PTR [rbp-0xe4],0x2b
   0x0000555555555239 <+187>:   mov    DWORD PTR [rbp-0xe0],0xc
   0x0000555555555243 <+197>:   mov    DWORD PTR [rbp-0xdc],0xe
   0x000055555555524d <+207>:   mov    DWORD PTR [rbp-0xd8],0xc
   0x0000555555555257 <+217>:   mov    DWORD PTR [rbp-0xd4],0x37
   0x0000555555555261 <+227>:   mov    DWORD PTR [rbp-0xd0],0xb
   0x000055555555526b <+237>:   mov    DWORD PTR [rbp-0xcc],0xb
--Type <RET> for more, q to quit, c to continue without paging--
   0x0000555555555275 <+247>:   mov    DWORD PTR [rbp-0xc8],0xa
   0x000055555555527f <+257>:   mov    DWORD PTR [rbp-0xc4],0xa
   0x0000555555555289 <+267>:   mov    DWORD PTR [rbp-0xc0],0x6
   0x0000555555555293 <+277>:   mov    QWORD PTR [rbp-0x20],0x19
   0x000055555555529b <+285>:   mov    rdx,QWORD PTR [rip+0x2d8e]        # 0x555555558030 <stdin@GLIBC_2.2.5>
   0x00005555555552a2 <+292>:   lea    rax,[rbp-0xb0]
   0x00005555555552a9 <+299>:   mov    esi,0x80
   0x00005555555552ae <+304>:   mov    rdi,rax
   0x00005555555552b1 <+307>:   call   0x555555555060 <fgets@plt>
   0x00005555555552b6 <+312>:   lea    rax,[rbp-0xb0]

```

At this point the program initializes a bunch of local variables and print `ECOWAS ADMIN PORTAL :` and wait for our input.
Then compare the length of our input with the value 25 at `[rbp-0x20]` and display `Flag wrong. try again` if the length is not `0x19`, 25 in `decimal`

```
┌──(kali㉿kali)-[~/CTFs/HackerlabCTF/Basic/Ecowas]
└─$ ./ecowas_portal                           
ECOWAS ADMIN PORTAL : 
test
Flag wrong. Try again. 
```

If the lenght is 25 the program and the flag is wrong the program display `Check failed`

```
┌──(kali㉿kali)-[~/CTFs/HackerlabCTF/Basic/Ecowas]
└─$./ecowas_portal                                                                                                      ECOWAS ADMIN PORTAL : 
AAAAAAAAAAAAAAAAAAAAAAAAA        
Check failed   
```
Let's find out what this check is

We know that our flag starts with `CTF_`, let's do some test in `gdb` with `CTF_AAAAAAAAAAAAAAAAAAAAA`.
we will go through our program instruction by instruction and understand how it works.

Let's execute the program in gdb with a `breakpoint` on `main` and browse the flow with the instruction `ni`, shortcut for `next instruction`. When the program asks for it, we put our crafted flag.

```
(gdb) 
0x00005555555552b1 in main ()
(gdb) 
CTF_AAAAAAAAAAAAAAAAAAAAA
0x00005555555552b6 in main ()
(gdb) 

```
Let's continue with our next instruction command
At this point the program subtracts `1` from the value of rax(the result of the strlen() function), compares rax to rbx `0` then jump if below. it is clearly a loop that will iterate until 25 (the length of our flag)

```
(gdb) info registers 
rax            0x1a                26
rbx            0x0                 0
rcx            0x0                 0
rdx            0xfffbfefefc000000  -1127003780546560
rsi            0x5555555596b1      93824992253617
rdi            0x7fffffffde30      140737488346672
rbp            0x7fffffffdee0      0x7fffffffdee0
rsp            0x7fffffffddc0      0x7fffffffddc0
r8             0xfefe              65278
r9             0x7ffff7fa1c00      140737353751552
r10            0xfffffffffffffb87  -1145
r11            0x7ffff7e70d80      140737352502656
r12            0x555555555080      93824992235648
r13            0x0                 0
r14            0x0                 0
r15            0x0                 0
rip            0x555555555361      0x555555555361 <main+483>
eflags         0x202               [ IF ]
cs             0x33                51
ss             0x2b                43
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0
(gdb) x/3i $rip
=> 0x555555555361 <main+483>:   sub    rax,0x1
   0x555555555365 <main+487>:   cmp    rbx,rax
   0x555555555368 <main+490>:   jb     0x5555555552f6 <main+376>
(gdb) 

```
Let's continue with our next instruction command.
then the program calls an `encrypt` function to which it passes two parameters `esi` and `edi`

```
0x0000555555555312 <+404>:   movsx  eax,al
0x0000555555555315 <+407>:   mov    edx,DWORD PTR [rbp-0x14]
0x0000555555555318 <+410>:   mov    esi,edx
0x000055555555531a <+412>:   mov    edi,eax
0x000055555555531c <+414>:   call   0x555555555169 <encrypt>
0x0000555555555321 <+419>:   mov    BYTE PTR [rbp-0x22],al
0x0000555555555324 <+422>:   movzx  eax,BYTE PTR [rbp-0x22]
0x0000555555555328 <+426>:   cmp    al,BYTE PTR [rbp-0x21]
0x000055555555532b <+429>:   je     0x555555555348 <main+458>

```
let's look at these parameters.

```
(gdb) p $esi
$5 = 0
(gdb) p $edi
$6 = 67
(gdb) x/2i $rip
=> 0x55555555531c <main+414>:   call   0x555555555169 <encrypt>
   0x555555555321 <main+419>:   mov    BYTE PTR [rbp-0x22],al
(gdb) 

```

```
└─$ python               
>>> chr(67)
'C'
>>> 
```
The program passes `$edi=67` and `$esi=0` to the encrypt function. Let's take a closer look at this function.

The function encrypt subtracts `0x14`, 20 in decimal from the value of `edi` and `xor` the result with the value of `esi`
```
(gdb) disass encrypt
Dump of assembler code for function encrypt:
   0x0000555555555169 <+0>:     push   rbp
   0x000055555555516a <+1>:     mov    rbp,rsp
   0x000055555555516d <+4>:     mov    DWORD PTR [rbp-0x4],edi
   0x0000555555555170 <+7>:     mov    DWORD PTR [rbp-0x8],esi
   0x0000555555555173 <+10>:    mov    eax,DWORD PTR [rbp-0x4]
   0x0000555555555176 <+13>:    sub    eax,0x14
   0x0000555555555179 <+16>:    xor    eax,DWORD PTR [rbp-0x8]
   0x000055555555517c <+19>:    pop    rbp
   0x000055555555517d <+20>:    ret    
End of assembler dump.

```
Let's take a closer look at what happens after the encrypt function is executed.
The return value of the encrypt function is stored in the `$rax` register, a byte (value at `al` register) of this value is copied to the memory address `[rbp-0x22]` and is compared to a byte of the value at the memory address `[rbp-0x21]`. if the values are equal then the program flow continues otherwise we get the message `check failed`.

```
0x000055555555531c <+414>:   call   0x555555555169 <encrypt>
0x0000555555555321 <+419>:   mov    BYTE PTR [rbp-0x22],al
0x0000555555555324 <+422>:   movzx  eax,BYTE PTR [rbp-0x22]
0x0000555555555328 <+426>:   cmp    al,BYTE PTR [rbp-0x21]
0x000055555555532b <+429>:   je     0x555555555348 <main+458>
```
At this stage the values are equal and our program can continue its execution.
```
(gdb) p/x $al
$12 = 0x2f

(gdb) x/b $rbp-0x21
0x7fffffffdebf: 0x2f
```

Let's start again from the beginning but this time with breakpoints
Let's place the first breakpoint at the input of the encrypt function and the second break before the jump if equal at the exit of the encrypt function.

let's launch the execution of our program with our crafted flag and at each loop let's eximinate the values of the `esi` and `edi` registers at the first break, then the values of the `al` register and the memory address `[rbp-0x21]` at the second break 

```
(gdb) break *0x000055555555531c
Breakpoint 3 at 0x55555555531c
(gdb) break *0x000055555555532b
Breakpoint 4 at 0x55555555532b
(gdb) run
Starting program: /home/kali/CTFs/HackerlabCTF/Basic/Ecowas/ecowas_portal 
ECOWAS ADMIN PORTAL : 
CTF_AAAAAAAAAAAAAAAAAAAAA

gdb) c
Continuing.
Breakpoint 3, 0x000055555555531c in main ()
(gdb) p $esi
$2 = 0
(gdb) p $edi
$3 = 67
(gdb) c
Continuing.

Breakpoint 4, 0x000055555555532b in main ()
(gdb) x/b $rbp-0x21
0x7fffffffdebf: 0x2f
(gdb) p/x $al
$4 = 0x2f
(gdb) c
Continuing.

Breakpoint 3, 0x000055555555531c in main ()
(gdb) p $esi
$5 = 1
(gdb) p $edi
$6 = 84
(gdb) c
Continuing.

Breakpoint 4, 0x000055555555532b in main ()
(gdb) x/b $rbp-0x21
0x7fffffffdebf: 0x41
(gdb) p/x $al
$7 = 0x41
(gdb) c
Continuing.

Breakpoint 3, 0x000055555555531c in main ()
(gdb) p $esi
$8 = 2
(gdb) p $edi
$9 = 70
(gdb) c
Continuing.

Breakpoint 4, 0x000055555555532b in main ()
(gdb) p/x $al
$10 = 0x30
(gdb) x/b $rbp-0x21
0x7fffffffdebf: 0x30

Breakpoint 3, 0x000055555555531c in main ()
(gdb) p $esi
$11 = 3
(gdb) p $edi
$12 = 95
(gdb) c
Continuing.

Breakpoint 4, 0x000055555555532b in main ()
(gdb) p/x $al
$13 = 0x48
(gdb) x/b $rbp-0x21
0x7fffffffdebf: 0x48
(gdb) 


```
Have you noticed anything?
Yes, that's exactly it. At each iteration, one character of our flag and the loop counter are passed to the encrypt function. 
It's not over yet.
Look at the values in the al register and the bunch of variables I mentioned at the beginning.
Yes, the byte of the value at memory address [rbp-0x21] to which the value of the al register is compared at each iteration is indeed one of these values.
```
└─$ python 
>>>
>>> chr(67)
'C'
>>> chr(84)
'T'
>>> chr(70)
'F'
>>> chr(95)
'_'
>>> 
```

let's summarize

The program declares and initializes a bunch of variables, then it asks for a user input `(the 25 characters flag)`, then it passes character by character the flag to a function `encrypt` which `subtracts 0x14` from each character of the flag and `xor` the result with the `counter value of the loop`. The result of this function is then compared in turn to the variables previously initialized by the program. If there is a match the program continues to the second character otherwise the program stops with the message `check failed`.


Now that we know how the program works, we can reverse it. 
I have written a python program to do this.

I converted to decimal the first 25 variables that our program initializes that I stored in a list then I browse the list by doing the reverse of what the encrypt function does.

```
#!/usr/bin/env python3
from pwn import xor

variables = [47,65,48,72,74,37,39,26,39,87,21,73,16,45,17,43,12,14,12,55,11,11,10,10,6]
flag=''

for i in range(len(variables)):
   flag=flag+chr(ord(xor(variables[i],i))+20)
   i = i+1
print(flag)
```
```
└─$ python reverse.py       
CTF_b451Cr3V0438032832012

```
Done!!!


