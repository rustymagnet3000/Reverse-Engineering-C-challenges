# Phoenix Stack2
## Static Analysis
##### Analyse the binary
`r2 -A stack_three`
##### Familiarity
```
[0x000104a8]> i
file     stack-two
format   elf
arch     arm
endian   little
```
##### Imports
```
[0x0001049c]> ii
[Imports]
   1 0x0001043c  GLOBAL    FUNC printf
   4 0x00010448  GLOBAL    FUNC gets
   5 0x00010454  GLOBAL    FUNC puts
   7 0x00010460  GLOBAL    FUNC fflush
   9 0x0001046c    WEAK  NOTYPE __deregister_frame_info
  16 0x00010478  GLOBAL    FUNC exit
```
##### Exports
```
[0x0001049c]> iE
[Exports]
084 0x000005ec 0x000105ec GLOBAL   FUNC  136 main
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
##### Interesting Strings
```
[0x0001049c]> fs strings;f
0x00010680 68 str.Congratulations__you_ve_finished_phoenix_stack_three_:___Well_done
0x000106c4 76 str.Welcome_to_phoenix_stack_three__brought_to_you_by_https:__exploit.education
0x00010710 31 str.calling_function_pointer____p
0x00010730 63 str.function_pointer_remains_unmodified_:___better_luck_next_time
```
##### Stack Variables
```
[0x000105ec]> pd 5 @main
|           ;-- $a:
/ (fcn) main 120
|   main (FILE *stream);
|           ; var int local_54h @ fp-0x54
|           ; var int local_50h @ fp-0x50
|           ; var int local_8h @ fp-0x8
|           ; arg FILE *stream @ r3
```
