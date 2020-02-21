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

#### __isoc99_scanf
There was no `scanf` signature.  Just a function pointer to `scanf`. I guessed this was related to using the `Global Offset Table.`

added
[0x00001090]> ? 0x7b1
hex     0x7b1
int32   1969

```
let a = "Don't patch it!"
print(a.count)
"15\n"
```

#### Code stuff

```
char
char *local_ptr_to_buffer;
```
![debugger_view](/images/2020/02/debugger-view.png)



#### The Magic Line
The `*` means it is adjusting the `buffer` and not the pointer.
```
*ptr_buffer = *ptr_buffer + ((char)((int)growing_num / 10) * '\n' - (char)growing_num);
```




```
seed	13783
seed	30945
seed	20007
seed	8977
seed	62839
seed	46657
seed	64455
seed	57969
seed	12567
seed	22433
seed	25959
seed	50641
seed	26807
seed	56577
seed	2823
Obfs str: Gtu.}'uj{fq!p{$
buffer: Don't patch it!
```
#### Writing the De-Obfuscater function
```
char * string_deobfuscator(char *ptr_buffer){

    uint seed = 0x7b1;
    while (*ptr_buffer != '\0') {
        seed = seed * 7 & 0xffff;
        printf("Char being changed\tChar:%c\tDecimal:%d\n",*ptr_buffer, *ptr_buffer);
        printf("Value added to char:\t%d\n\r",((seed / 10) * '\n' - seed));
        *ptr_buffer = *ptr_buffer + ((seed / 10) * '\n' - seed);

        // increment a char along in the buffer. This is how it moves a single char at a time..
        ptr_buffer = ptr_buffer + 1;
    }

    return ptr_buffer;
}

int main (void) {
    char buffer [264];
    char *obfs_str = "Gtu.}\'uj{fq!p{$";             // = "Don't patch it!"
    strcpy(buffer,obfs_str);
    char *ptr_buffer = buffer;

    string_deobfuscator(ptr_buffer);

    printf("Obfs str: %s\n", obfs_str);
    printf("buffer: %s\n", buffer);
    return 0;
}
```


#### De-obfuscating Strings
const char* const obfs_strings[] = { "Gtu.}'uj{fq!p{$",
                                    "Lszl{{%",
                                    "vx{!whvt|twg?%",
                                    "%64[^\n]",
                                    "fhz4yhx|~g=5",
                                    "Ftyynjy*",
                                    "Zwvup("};

char * string_deobfuscator(char *ptr_buffer){

    uint seed = 0x7b1;
    while (*ptr_buffer != '\0') {
        seed = seed * 7 & 0xffff;
        *ptr_buffer = *ptr_buffer + ((seed / 10) * '\n' - seed);
        // increment a char along in the buffer. This is how it moves a single char at a time..
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

#### Answer
```
$ ./login-cipher
Don't patch it!
Insert your password: ccs-passwd44
Correct!
```
