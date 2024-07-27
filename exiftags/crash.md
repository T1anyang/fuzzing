# Bug 1

In function `parsetag`, there is a heap overflow in exiftags 1.01.  `strncpy` function can read illegal address by setting `prop->value`.

``` sh
./exiftags crash.jpeg

=================================================================
==1767622==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x61d0000008b4 at pc 0x5555555f0e01 bp 0x7fffffffddc0 sp 0x7fffffffd580
READ of size 30 at 0x61d0000008b4 thread T0
    #0 0x5555555f0e00 in __interceptor_strncpy (/home/yty/exiftags/exiftags+0x9ce00) (BuildId: 49c6389a011dbf2be2adbe22b105cdf590ffde71)
    #1 0x555555644208 in parsetag /home/yty/exiftags/exif.c:616:3
    #2 0x555555644208 in readtag /home/yty/exiftags/exif.c:177:2
    #3 0x555555644208 in readtags /home/yty/exiftags/exif.c:216:3
    #4 0x555555644208 in exifscan /home/yty/exiftags/exif.c:882:3
    #5 0x555555645084 in exifparse /home/yty/exiftags/exif.c:902:12
    #6 0x555555642726 in doit /home/yty/exiftags/exiftags.c:152:7
    #7 0x555555642465 in main /home/yty/exiftags/exiftags.c:280:8
    #8 0x7ffff7ceac89  (/lib/x86_64-linux-gnu/libc.so.6+0x27c89) (BuildId: 2e01923fea4ad9f7fa50fe24e0f3385a45a6cd1c)
    #9 0x7ffff7cead44 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x27d44) (BuildId: 2e01923fea4ad9f7fa50fe24e0f3385a45a6cd1c)
    #10 0x5555555824a0 in _start (/home/yty/exiftags/exiftags+0x2e4a0) (BuildId: 49c6389a011dbf2be2adbe22b105cdf590ffde71)

0x61d0000008b4 is located 0 bytes to the right of 2100-byte region [0x61d000000080,0x61d0000008b4)
allocated by thread T0 here:
    #0 0x555555605eb2 in __interceptor_malloc (/home/yty/exiftags/exiftags+0xb1eb2) (BuildId: 49c6389a011dbf2be2adbe22b105cdf590ffde71)
    #1 0x5555556426aa in doit /home/yty/exiftags/exiftags.c:141:30

SUMMARY: AddressSanitizer: heap-buffer-overflow (/home/yty/exiftags/exiftags+0x9ce00) (BuildId: 49c6389a011dbf2be2adbe22b105cdf590ffde71) in __interceptor_strncpy
Shadow bytes around the buggy address:
  0x0c3a7fff80c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c3a7fff80d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c3a7fff80e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c3a7fff80f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c3a7fff8100: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c3a7fff8110: 00 00 00 00 00 00[04]fa fa fa fa fa fa fa fa fa
  0x0c3a7fff8120: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c3a7fff8130: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c3a7fff8140: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c3a7fff8150: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c3a7fff8160: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==1767622==ABORTING
```