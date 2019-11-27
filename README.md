# Explore vulnerable C APIs
## Overview
The Internet is full of excellent ( and poor ) references to vulnerabilities in C and C++.  In 2019, C-related vulnerabilities were still voted the number one issue in Software Security.  The top 25 issues were listed [here](https://cwe.mitre.org/).

Most of the APIs are relatively well-known:

- gets
- strcpy
- printf
- sprintf
- scanf
- access (symbolic link)

### Reverse Engineering C challenges
You can try and use these insecure C APIs to complete online challenges.  Most offer a Virtual Machine so you play without worry.

```
https://exploit.education/
https://azeria-labs.com/part-3-stack-overflow-challenges/
https://samsclass.info/
https://github.com/pedromigueladao/SSof/wiki/BufferOverflows
```
### Run vulnerable code on macOS?
Most C APIs have been around for 40+ years.  People have worked to stop even beginner developers hitting these vulnerabilities.  How?

 - On macOS `Clang` swaps out insecure APIs for more secure variants of the same API
 - Again `Clang` adds lots of `Stack Guards` and `Memory Guards` to detect and stop overflows
 - Almost all Development Tools `Warn of insecure API usage` at compile time
 - When in debug mode, even the runtime will print `warning` of insecure API usage
 - `Static code analysis` tools will report on usage of vulnerable C APIs

### Explore on macOS
You need to remove a mountain of default compiler flags, if you compile vulnerable C code with Xcode.  Why use Xcode?  I like auto-complete when coding.  Plus, you won't make tiny mistakes that you can't avoid, when using a text editor to code.  Besides, it is not 1985.

### Understanding macOS
When using C APIs on macOS, it is useful to get familiar with `Lazy loading`, `stub_helpers` and the `dynamic linker`.  The following articles cover all of these topics:
```
https://adrummond.net/posts/macho
https://www.reinterpretcast.com/hello-world-mach-o
```
Here are some direct quotes:
> By default, dynamically bound symbols like printf are bound **lazily**. That is, printf is not bound when the executable is loaded, but only when the first call to printf is made.
>
> The **stub helper** calls the **dynamic linker**. The dynamic linker overwrites the lazy symbol pointer with the address of the printf function itself. All arguments to dyld_stub_binder are passed on the stack, so the arguments to printf aren't clobbered.

### Clang Help
You will need help to understand the Compiler flags and the C APIs.  macOS ships with `Clang`:
```
clang -help | grep -i ffree
```
and you can also use..
```
man strcpy
```
### Compiler Security flags
```
-ffreestanding          
Assert that the compilation takes place in a freestanding environment

-fno-stack-check
Disable stack checking

-fno-sanitize=address
Turns off AddressSanitizer; used to detect memory corruption bugs such as buffer overflows or a dangling pointer (use-after-free)

-fno-builtin
Disable implicit builtin knowledge of functions

-fno-stack-protector
Disable the use of Stack Canaries

-fno-PIE
Turn off ASLR

-fno-fsanitize=cfi
Disable control flow integrity (CFI) checks

-fno-delete-null-pointer-checks
Do not treat usage of null pointers as undefined behavior

-fno-autolink
Disable generation of linker directives for automatic library linking

-g
For Debug symbols, to simplify debugging
```
### Insecurely Compile
```
clang -fno-sanitize=address -ffreestanding -fno-stack-check -fno-builtin -fno-stack-protector -D_FORTIFY_SOURCE=0 -fno-PIE code.c -o tryme
```
Even with this, macOS will use the `stubHelper` to replace the insecure calls with Secure Calls.

Depending on what flags are on / off, you could see:
```
EXC_BAD_ACCESS (code=EXC_I386_GPFLT)
EXC_BAD_INSTRUCTION (code=EXC_I386_INVOP, subcode=0x0)
```

### Compile and Run
```
// Example: https://wiki2.org/en/AddressSanitizer

int global_array[100] = {-1};
int main(int argc, char **argv) {
  return global_array[argc + 100];  // Overflow by the number of arguments passed into the app
}

// Detects Overflow
clang -O -g -fsanitize=address vuln.c && ./a.out

// Does not detect Overflow
clang -O -g -fno-sanitize=address vuln.c && ./a.out
```

### References
```
https://security.web.cern.ch/security/recommendations/en/codetools/c.shtml
https://wiki2.org/en/Buffer_overflow_protection#Clang/LLVM
https://cwe.mitre.org/data/definitions/120.html
https://stackoverflow.com/questions/35659152/stack-based-buffer-overflow-challenge-in-c-using-scanf-with-limited-input

```
