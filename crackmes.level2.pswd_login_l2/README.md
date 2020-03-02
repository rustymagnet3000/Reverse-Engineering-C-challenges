## https://crackmes.one/

Name | Password Login 2
--|--
Author  | Loz
Level  |  2

Author comment:
```
No patching allowed, figure out the password, hopefully the difficulty is appropriate
```
#### Run
```
pswd_login_l2
aaa
Login failed
```
#### Registers
I found, that I leaned on registers heavily in this crackme. As a reminder:
```
First Argument: RDI
Second Argument: RSI
Third Argument: RDX
Fourth Argument: RCX
Fifth Argument: R8
Sixth Argument: R9
Return Value: RAX
```

#### Symbols
Lots of C++ Symbols. Much bigger than any of the challenges to date. Cut down list below:
```
[0x00002280]> is~FUNC
050 0x00002800 0x00002800   WEAK   FUNC   55 password::wrongPassword()
054 0x00002570 0x00002570   WEAK   FUNC  171 password::checkLength(int)
055 0x0000261c 0x0000261c   WEAK   FUNC  484 password::checkPassword(std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>)
062 0x00002870 0x00002870   WEAK   FUNC  110 password::password()
070 0x00002365 0x00002365 GLOBAL   FUNC  305 main
081 0x00002838 0x00002838   WEAK   FUNC   55 password::rightPassword()
008 0x000020a0 0x000020a0 GLOBAL   FUNC   16 imp.memcmp
012 0x00000000 0x00000000 GLOBAL   FUNC   16 imp.vsnprintf
```
#### Class of interest

![Password_class](/images/2020/02/password-class.png)

#### Information
```
rabin2 -I pswd_login_l2
arch     x86
canary   false
class    ELF64
lang     c
os       linux
stripped false
```
#### Strings
```
rabin2 -qz pswd_login_l2
0x300c 13 12 Login failed
0x3019 17 16 Login successful
0x302a 19 18 x_.1:.-8.4.p6-e.!-
0x3040 42 41 basic_string::_M_construct null not valid
```
#### Getting past the CheckLength function
Ghidra - with some `variable renaming` - looked like this:

![check_length_fn](/images/2020/02/check-length-fn.png)


As you would expect, a debugger `Breakpoint` showed the First Register (`RDI`) was set to **5** when I entered `AAAAA`.

But the Ghidra de-compile showed `casts` between `int` and `char`. This made the True/False check trickier to read.  `radare2` de-compiled without this. It was cleaner to read.  In short, Ghidra often de-compiled with:

```
// Ghidra
(char) $1 = '\x01'    TRUE
(char) $0 = '\0'      FALSE
```
#### A C++ rat hole
I noticed a function I didn't recognize. In real C++ code it was this API:
```
str0 = "C+_ String 0";
str0.at(2) = '+';
```
Setting the breakpoint I wanted to inspect the value passed into the function and the character being assigned to this position in the string.
```
gef➤  br at
$rsi   : 0x1		-> setting a char in space 1

gef➤  p $cs
$7 = 0x33		-> 0x33 is the char '!'
```
I then tried a couple of guesses:
```
./pswd_login_l2
A!AAAA
Login failed

./pswd_login_l2
x!.1:.-8.4.p6-e.!-
Login failed
```
#### Brute Force the Password
Note to self -> when you don't understand all of the language `Symbols`, fall back on the `Registers`.  

I was going into rat holes looking at C++ APIs.  When all I needed was faith the `length` of the stored password was passed into a register.

In Ghidra, I could not spot the length of the stored password.  

I had a hunch the password was the hardcoded string:
```
.-8.4.p6-e.!-_0010302a

>>> len('x_.1:.-8.4.p6-e.!-')
18
```
I considered writing a debugger script to brute-force guess the password length.

Before that, I wanted to see if I could set a breakpoint on a function that did not fire, unless the length was correct.

```
b _ZN8password13checkPasswordENSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
```
De-mangled:
```
password::checkPassword(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)
```
That instantly got me a result:
```
gef➤  r
Starting program: /home/user/pswd_login_l2
ABCDEFG
```
Now I knew the password length was 7.

#### Check Password
Finding the Seed Value for XOR
```
0x55555555669a <password::checkPassword ..> 	xor    eax, edx

b *0x000055555555669a

// my entered Password(char)
gef➤  p $edx
$11 = 0x41

// always a `B` character
gef➤  p $eax
$12 = 0x42

```
