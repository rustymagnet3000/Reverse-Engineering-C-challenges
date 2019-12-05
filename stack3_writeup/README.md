# Protostar Stack1
## Static Analysis
##### Analyse the binary
`r2 -A stack3`
##### Imports
```
[0x08048370]> ii
[Imports]
   1 0x08048320    WEAK  NOTYPE __gmon_start__
   2 0x08048330  GLOBAL    FUNC gets
   3 0x08048340  GLOBAL    FUNC __libc_start_main
   4 0x08048350  GLOBAL    FUNC printf
   5 0x08048360  GLOBAL    FUNC puts
```
##### Interesting Strings
```
0x08048540 31 str.code_flow_successfully_changed
0x08048560 45 str.calling_function_pointer__jumping_to_0x_08x
```
##### Stack Variables
```
[0x08048370]> pdf @main
|           ;-- main:
/ (fcn) sym.main 65
|   sym.main ();
|           ; var unsigned int local_4h @ esp+0x4
|           ; var char *s @ esp+0x1c
|           ; var unsigned int local_5ch @ esp+0x5c
```
