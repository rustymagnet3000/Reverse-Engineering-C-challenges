## https://crackmes.one/

Name | iso_32
--|--
Author  | math0x0
Level  |  3

#### Run
This binary won't run natively on a 64-bit Virtual Machine, unless you install additional packages.
```
./math0x0
enter the password:aaa
Ooop!! Try again
```
#### Binary Info
```
rabin2 -I math0x0
arch     x86
binsz    14295
bintype  elf
bits     32
stripped false
```
#### Strings
![strings_math0x0](/images/2020/03/strings-math0x0.png)
#### Symbols
It looked like the author purposely tried to hide the meaning of the function names.  The binary was not stripped of debug Symbols.
```
rabin2 -s math0x0
[Symbols]

030 0x00003024 0x0804c024  LOCAL    OBJ    1 completed.6972
049 0x000011d8 0x080491d8 GLOBAL   FUNC   64 f
050 0x000011ad 0x080491ad GLOBAL   FUNC   43 __f_func
061 0x00002000 0x0804a000 GLOBAL    OBJ    4 _fp_hw
063 0x00001218 0x08049218 GLOBAL   FUNC   33 main
064 0x00001239 0x08049239 GLOBAL   FUNC    0 __x86.get_pc_thunk.ax
065 0x00001182 0x08049182 GLOBAL   FUNC   43 __s_func
imp.__isoc99_scanf
```
#### Main
![main_cleaned_up_math0x0](/images/2020/03/main-cleaned-up-math0x0.png)

#### No obvious Compare instruction
In Ghidra you could select `Search \ Program Text`. Then you could search for a `CMP` instruction to see if something obvious was being done.


![search_mnemonic](/images/2020/03/search-mnemonic.png)

#### scanf
Was this a `Buffer Overflow` challenge?  

![scanf_buffer_math0x0](/images/2020/03/scanf-buffer-math0x0.png)

Looks like it:
```
run <<< $(python3 -c 'print ("\x41" * 50)')
[#0] Id 1, stopped, reason: SIGSEGV
─────────────────
0x41414141 in ?? ()
```
I get a `Segmentation Fault` in the `Instructions Pointer ($eip)`.`
#### Debugger
After you enter a password is gets entered into the `Stack Pointer`.
```
$esp   : 0xffffd570  →  "AAAAAA"
```
