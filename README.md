# Explore vulnerable C APIs
## Overview
This repo includes the notes I took to solve the challenges from https://exploit.education/phoenix/ using vulnerable C APIs.  

I started the repo to remind myself of [ obscure ] commands while also teaching people who were learning to use a `debugger` or a  `disassembler`.

### Reverse Engineering C challenges
The Internet is full of excellent references to vulnerabilities in C.  In 2019, C-related vulnerabilities were still voted the number one issue in Software Security.  The [top 25 issues](https://cwe.mitre.org/).  Most of the APIs were relatively well known:

- gets
- strcpy
- printf
- sprintf
- scanf
- access

You can use these insecure APIs to complete online `Memory Corruption` challenges.  Most offer a Virtual Machine so you play without worry.
```
https://exploit.education/
https://www.hackthebox.eu/
https://www.virtualhackinglabs.com/
https://www.vulnhub.com/
https://azeria-labs.com/part-3-stack-overflow-challenges/
https://samsclass.info/
```
### Getting Started
There are some good references to getting a Virtual Machine setup.
```
http://www.iet.unipi.it/p.perazzo/teaching/cybsec/LAB.01.Phoenix_setup.pdf
https://blog.lamarranet.com/index.php/exploit-education-phoenix-setup/
```

### References
```
https://security.web.cern.ch/security/recommendations/en/codetools/c.shtml
https://wiki2.org/en/Buffer_overflow_protection#Clang/LLVM
https://cwe.mitre.org/data/definitions/120.html
https://stackoverflow.com/questions/35659152/stack-based-buffer-overflow-challenge-in-c-using-scanf-with-limited-input

```
