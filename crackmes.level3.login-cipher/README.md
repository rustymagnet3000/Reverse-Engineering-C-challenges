## https://crackmes.one/

Name | login-cipher
--|--
Author  |  Silva97
Level  |  3

#### Run
```
$ ./login-cipher
Don't patch it!
Insert your password: AAAAA
Wrong!
```
#### Symbols
No `main` function visible.
```
[0x00001090]> is~FUNC
002 0x00001030 0x00001030 GLOBAL   FUNC   16 imp.strcpy
003 0x00001040 0x00001040 GLOBAL   FUNC   16 imp.puts
004 0x00001050 0x00001050 GLOBAL   FUNC   16 imp.__stack_chk_fail
005 0x00001060 0x00001060 GLOBAL   FUNC   16 imp.fputs
006 0x00000000 0x00000000 GLOBAL   FUNC   16 imp.__libc_start_main
008 0x00001070 0x00001070 GLOBAL   FUNC   16 imp.__isoc99_scanf
011 0x00000000 0x00000000   WEAK   FUNC   16 imp.__cxa_finalize
006 0x00000000 0x00000000 GLOBAL   FUNC   16 imp.__libc_start_main
011 0x00000000 0x00000000   WEAK   FUNC   16 imp.__cxa_finalize
```
#### Stripped and Stack Guards
```
[0x00001090]> i
file     login-cipher
humansz  14.0K
type     DYN (Shared object file)
arch     x86
canary   true
stripped true
```
#### Strings
All strings were `Obfuscated`.
```
crackme rabin2 -qz login-cipher    
0x2004 16 15 Gtu.}'uj{fq!p{$
0x2014 8 7 Lszl{{%
0x201c 15 14 vx{!whvt|twg?%
0x202b 8 7 %64[^\n]
0x2033 13 12 fhz4yhx|~g=5
0x2040 9 8 Ftyynjy*
0x2049 7 6 Zwvup(
```
Ghidra missed the following obfuscated string. Radare2 got confused on the space in the String.

```
Clean:  "Insert your password:"
Obfs:   "Lszl{{% vx{!whvt|twg?%"
```

In Ghidra it was marked as a memory reference ( `&DAT_0010201` ). This address was the start of the string's bytes in memory.

#### Find main
There was no `main` function reference. But `__libc_start_main` was available.

> int libc_start_main(int *(main) (int, char * *, char * *), int argc, char * * ubp_av, void (*init) (void), void (*fini) (void), void (*rtld_fini) (void), void (* stack_end));

From that `signature` you could see the first pointer argument was to `main`.

#### Tidy-up Main
You cannot over-state the value of `renaming variables` and `renaming functions`.  Of course, you get it wrong. As you slowly unpick the code, you refine.  I eventually landed here:

![main_cleaned_up](/images/2020/02/main-cleaned-up.png)

#### Code the De-obfuscater
I still had a bunch of `obfuscated strings`.  I had no `plaintext password`.  What next?  I wanted to de-obfuscate all Strings.  I had good psuedo-code from  Ghidra.


![str_deobf](/images/2020/02/str-deobf.png)


#### Re-code and run the function
With a debugger, I finally understood:  

![debugger_view](/images/2020/02/debugger-view.png)

The code was subtle.  I don't think I would have picked the subtlety without this step.

```
*ptr_buffer = a character in the mutable Buffer.
ptr_buffer = the pointer to the character in the buffer. It was set to start at the first character in the buffer.
The pointer incremented on each loop.
```
#### The Obfuscation
The following line took the obfuscated character in the Buffer ( `*ptr_buffer` ) and subtracted a value.  

```
*ptr_buffer = *ptr_buffer + ((char)((int)growing_num / 10) * '\n' - (char)growing_num);
```
Removing the casts:
```
*ptr_buffer = *ptr_buffer + ((seed / 10) * '\n' - seed);
```
#### Known-plaintext attack
From Ghidra, and running the program, I knew that:
```
"Gtu.}'uj{fq!p{$"  == "Don't patch it!"
```
It was the `seed` value that create the apparent randomness. But it was a consistent pattern.  This was not random.  Back to the `ASCII Table`.

```
Obfuscated string: Gtu.}'uj{fq!p{$
Seed:13783	Value to subtract :-3	Original:G	New:D
Seed:30945	Value to subtract :-5	Original:t	New:o
Seed:20007	Value to subtract :-7	Original:u	New:n
Seed:8977	Value to subtract :-7	Original:.	New:'
Seed:62839	Value to subtract :-9	Original:}	New:t
Seed:46657	Value to subtract :-7	Original:'	New:
Seed:64455	Value to subtract :-5	Original:u	New:p
Seed:57969	Value to subtract :-9	Original:j	New:a
Seed:12567	Value to subtract :-7	Original:{	New:t
Seed:22433	Value to subtract :-3	Original:f	New:c
Seed:25959	Value to subtract :-9	Original:q	New:h
Seed:50641	Value to subtract :-1	Original:!	New:
Seed:26807	Value to subtract :-7	Original:p	New:i
Seed:56577	Value to subtract :-7	Original:{	New:t
Seed:2823	Value to subtract :-3	Original:$	New:!
Clean string: Don't patch it!
```
#### Answer
```
$ ./login-cipher
Don't patch it!
Insert your password: ccs-passwd44
Correct!
```


#### Code to Plaintext all Obfuscated Strings
```
const char* const obfs_strings[] = { "Gtu.}'uj{fq!p{$",
                                    "Lszl{{% vx{!whvt|twg?% ",
                                    "%64[^\n]",
                                    "fhz4yhx|~g=5",
                                    "Ftyynjy*",
                                    "Zwvup("};

char * string_deobfuscator(char *ptr_buffer){

    uint seed = 0x7b1;
    while (*ptr_buffer != '\0') {
        seed = seed * 7 & 0xffff;
        *ptr_buffer = *ptr_buffer + ((seed / 10) * '\n' - seed);
        ptr_buffer = ptr_buffer + 1;
    }

    return ptr_buffer;
}

int main (void) {
    char buffer [264];
    size_t elements_in_arr = sizeof(obfs_strings)/sizeof(obfs_strings[0]);

    for (int i = 0; i < elements_in_arr; i++) {
        printf("Obfuscated string: %s\n", obfs_strings[i]);
        strcpy(buffer,obfs_strings[i]);
        char *ptr_buffer = buffer;
        string_deobfuscator(ptr_buffer);
        printf("Clean string: %s\n\n", buffer);
    }

    return 0;
}
```
