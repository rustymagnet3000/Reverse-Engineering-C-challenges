# Exploring C / C++ vulnerabilities on macOS
## Overview
The Internet is full of excellent ( and poor ) references to vulnerabilities in C and C++.  

By a large distance, in 2019 C-related vulnerabilities were still voted the number one issue in Software security.  The top 25 is compelling reading [ https://cwe.mitre.org/ ], if you enjoy `Application Security`.

### Reality VS Noise
Most C APIs have been around for 40+ years.  People have worked to stop even beginner developers from hitting these vulnerabilities.  How?

Warnings...
 - Almost all Development Tools `Warn of insecure API usage` at compile time
 - When in debug mode, even the runtime will print `warning` of insecure API usage.
 - `Static code analysis` tools will report on usage of vulnerable C APIs.  

Swapping out your code, by default...

 - On macOS `Clang` uses it Compiler and Linker to ensure it swaps out insecure APIs for more secure variants of the same API.
 - Again `Clang` adds lots of `Stack Guards` and `Memory Guards` to detect and stop overflows.

### Start
You need to remove a mountain of default compiler flags, if you compile vulnerable C code with Xcode.  Why use Xcode?  I like auto-complete when coding.  Plus, you won't make tiny mistakes that you can't avoid, when using a text editor to code.  Besides, it is not 1985.

### Understanding macOS
When using C APIs on macOS, it is useful to get familiar with `Lazy loading`, `stub_helpers` and the `dynamic linker`.  The following articles cover all of these topics:
```
https://adrummond.net/posts/macho
https://www.reinterpretcast.com/hello-world-mach-o
```
Here are some direct quotes:
- By default, dynamically bound symbols like printf are bound *lazily*. That is, printf is not bound when the executable is loaded, but only when the first call to printf is made.
- The stub helper calls the dynamic linker. The dynamic linker overwrites the lazy symbol pointer with the address of the printf function itself. All arguments to dyld_stub_binder are passed on the stack, so the arguments to printf aren't clobbered.

### Help
You will need help to understand the Compiler flags and the C APIs.  macOS ships with `Clang`:
```
clang -help | grep -i ffree
```
and you can also use..
```
man strcpy
```
These are the flags I used:
```
-ffreestanding          
Assert that the compilation takes place in a freestanding environment

-fno-stack-check
Disable stack checking

-fno-builtin
Disable implicit builtin knowledge of functions

-fno-stack-protector
Disable the use of stack protectors

-fno-PIE
Only required to better show the power of the exploit

-fno-fsanitize=cfi

-fno-delete-null-pointer-checks
Do not treat usage of null pointers as undefined behavior.

-fno-autolink
Disable generation of linker directives for automatic library linking

-g
For Debug symbols, to make examples simpler to understand
```

### References
```
https://security.web.cern.ch/security/recommendations/en/codetools/c.shtml

https://cwe.mitre.org/data/definitions/120.html
```
