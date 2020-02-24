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
Ghidra with some variable renaming looked like this:
```
pswdBoolean = checkLength(enteredPassword,len);
```
A Breakpoint showed the First Register (`RDI`) was set to **5** when I entered "AAAAA".

Disassembled the `mangled function`:

`disas _ZN8password11checkLengthEi`

The return value was `0x0`.

More usefully, a breakpoint on a C++ String initialization failed to fire.
```
call   0x5555555560b0 <_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEED1Ev@plt>
```

#### Registers
```
First Argument: RDI
Second Argument: RSI
Third Argument: RDX
Fourth Argument: RCX
Fifth Argument: R8
Sixth Argument: R9
Return Value: RAX
```

#### C APIs
```
// get Password input
getline(string buffer, max length);

// convert String to Int
atoi(String)

// Set a command to fire at a certain time
at(....)


>>> len('x_.1:.-8.4.p6-e.!-')
18
