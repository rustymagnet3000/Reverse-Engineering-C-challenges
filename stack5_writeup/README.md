# Phoenix Stack4

##### Result
This challenge started with a question:

> As opposed to executing an existing function in the binary, this time we’ll be introducing the concept of “shell code”, and being able to execute our own code.


##### Learning
##### Analysis
```
// set breakpoint after gets()
b *0x00010500

gef> p/u $fp - $sp
$1 = 132

[0x000103b4]> pdf @0x000104e8
/ (fcn) sym.start_level 36
|           0x000104f0      80d04de2       sub sp, sp, 0x80   <-- space for local var
|           0x000104f4      84304be2       sub r3, fp, 0x84   <-- same as above
|           0x000104f8      0300a0e1       mov r0, r3                  ; char *s
|           0x000104fc      9dffffeb       bl sym.imp.gets             ; char *gets(char *s)

```

The space given at the start of the `start_level` function looks like a `128 char buffer` and a `4 char pointer`.  That makes sense if you look at `man gets`.  `gets` returns a pointer.

```
LIBRARY
     Standard C Library (libc, -lc)

SYNOPSIS
     #include <stdio.h>

     char *
     gets(char *str);
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
