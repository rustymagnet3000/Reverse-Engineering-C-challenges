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
$ ./pswd_login_l2
aaa
Login failed
```
#### Registers
When you don't understand all of the language `Symbols`, you can fall back on the `CPU Registers`.  I leaned on registers heavily in this crackme:
```
Return Value: RAX
First Argument: RDI
Second Argument: RSI
Third Argument: RDX
Fourth Argument: RCX
Fifth Argument: R8
Sixth Argument: R9
```

#### Symbols
Lots of C++ Symbols. The Class of interest was `de-mangled` by Ghidra:

![Password_class](/images/2020/02/password-class.png)

#### Information
```
rabin2 -I pswd_login_l2
arch     x86
endian   little
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
I was going into rat holes looking at C++ APIs.  When all I needed was faith the `length` of the stored password was passed in `register`.  In Ghidra, I could not spot the length of the stored password.  

I had a hunch the password was the hardcoded string:
```
.-8.4.p6-e.!-_0010302a

>>> len('x_.1:.-8.4.p6-e.!-')
18
```
I considered writing a debugger script to brute-force guess the password length.  But there was a simpler solution; set a `breakpoint` on a function that would not trigger unless the length was correct.

Breakpoint was set on this de-mangled function:
```
password::checkPassword(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)
```
That instantly got me a breakpoint hit:
```
gef➤  r
Starting program: /home/user/pswd_login_l2
ABCDEFG
```
Now I knew the password length was **7**.

#### Check Password - understanding the XOR
Inside this function, I could see `XOR` instructions.

I wanted to find the Seed Value that was being `XOR'd` with my entered password.
```
0x55555555669a <..checkPassword ..> xor eax, edx

b *0x000055555555669a

// my entered Password(char)
gef➤  p $edx
$11 = 0x41

// always a `B` character
gef➤  p $eax
$12 = 0x42
```
Now we know the XOR is always `'B'`.
#### Check Password - XOR hardcoded Strings
Inside `Script Manager` in `Ghidra` filter on `xor`.

![script_manager_ghidra](/images/2020/03/script-manager-ghidra.png)

Then just set the Key to `42`.

Although it was not relevant here [ as I only had a single hex byte ] the XOR Key Value needed to be written in the correct `Endian` order.

Setting that script, it auto-translated `'x_.1:.-8.4.p6-e.!-'` with the XOR value.  I didn't have to specify the memory address.

![auto_xord_string](/images/2020/03/auto-xord-string.png)

#### Check Password - narrowing in
The password is 7 characters of `:lsxlozlvl2to'lcoB`.  Which characters?  

![loop_in_ghidra](/images/2020/03/loop-in-ghidra.png)

Putting a breakpoint on `offset x2677` gave me the Loop index. If you continued, the `$eax` register was incrementing by one. This piece of code was a loop that `XOR'd` each character in my 7 character string.

That gave me a clue.  Look at the code after the loop.  At some point there would be a compare of strings in the `Registers`.  The listing View in Ghidra gave me the clue:
```
0010279e e8 ec 02        CALL       std::operator==<char>
```

A breakpoint triggered. Printing the **second function argument**:

```
$rsi   : 0x00007fffffffe310  →  0x00007fffffffe320  →  0x00702e342e382d2e (".-8.4.p"?)
```
This was the `Password` object. The first 16 bytes were the reference to `self` (`x310`).

```
gef➤  p/u 0x7fffffffe320 - 0x7fffffffe310
16
```
For a more useful example of the **first function argument** being compared to the **second function argument**:

```
r
Starting program:
lllllll
```
When the breakpoint triggers on the `operator==<char>` you can see:
```
First Argument: RDI
$rdi   : 0x00007fffffffe330  →  0x00007fffffffe340  →  0x002e2e2e2e2e2e2e (".......")
Second Argument: RSI
$rsi   : 0x00007fffffffe310  →  0x00007fffffffe320  →  0x00702e342e382d2e (".-8.4.p")
```
#### Answer
You can see with `gdb`, the call `operator==<char>` is  comparing 7-characters from the hardcoded string to the XOR result of the user entered input.

The `Password` object was on the `Heap`.  Searching for `Heap memory references` in Ghidra was useless.  Hence, sometimes you need to rely on multiple tools to solve a crackme.

```
$ ./pswd_login_l2
lozlvl2
Login successful
```
