# bug 1: stack overflow

In giftrans <=1.12.2, there is a stack overflow in function `giftrans`. `fread` in line 619 does not check variable `size`, it can be bigger than `sizeof(gce)`, which is 5. This bug can lead to DoS and RCE.

``` sh
./giftrans-asan -TLV crash.gif -o /dev/null
Header: "GIFA��"
Logical Screen Descriptor:
        Logical Screen Width: 42069 pixels
        Logical Screen Height: 20548 pixels
        Global Color Table Flag: True
        Color Resolution: 3 bits
        Sort Flag: False
        Size of Global Color Table: 8 colors
        Background Color Index: 32
Global Color Table:
        Color 0: Red 0, Green 0, Blue 237, #0000ed
        Color 1: Red 56, Green 55, Blue 97, #383761
        Color 2: Red 92, Green 1, Blue 32, #5c0120
        Color 3: Red 1, Green 247, Blue 71, #01f747
        Color 4: Red 73, Green 70, Blue 37, #494625
        Color 5: Red 57, Green 71, Blue 73, #394749
        Color 6: Red 70, Green 117, Blue 0, #467500
        Color 7: Red 16, Green 0, Blue 0, #100000
Unknown label: 0x10
Comment Extension
        \004GIF87a\\001\246H\001\001\000!\371\004\002!\376\010\000\306\000, \012=\000\000\010\377\320>*\210@\207;\001\000\000!\371\004\002\010\000\306\000, \000!\376<\004GIF
Image Descriptor:
        Image Left Position: 32 pixels
        Image Top Position: 61 pixels
        Image Width: 2048 pixels
        Image Height: 53503 pixels
        Local Color Table Flag: False
        Interlace Flag: False
Local Color Table:
        Color 0: Red 42, Green 1, Blue 1
        Color 1: Red 1, Green 1, Blue 1
Table Based Image Data:
        LZW Minimum Code Size: 0xf9
Image Descriptor:
        Image Left Position: 32 pixels
        Image Top Position: 61 pixels
        Image Width: 2048 pixels
        Image Height: 53503 pixels
        Local Color Table Flag: False
        Interlace Flag: False
Local Color Table:
        Color 0: Red 42, Green 72, Blue 1
        Color 1: Red 1, Green 0, Blue 33
Table Based Image Data:
        LZW Minimum Code Size: 0xf9
Image Descriptor:
        Image Left Position: 247 pixels
        Image Top Position: 61 pixels
        Image Width: 2048 pixels
        Image Height: 53503 pixels
        Local Color Table Flag: False
        Interlace Flag: False
Local Color Table:
        Color 0: Red 42, Green 136, Blue 64
        Color 1: Red 126, Green 74, Blue 1
Table Based Image Data:
        LZW Minimum Code Size: 0x00
Graphic Control Extension:
=================================================================
==2717495==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7fffffff9a85 at pc 0x555555595182 bp 0x7fffffff9330 sp 0x7fffffff8b00
WRITE of size 105 at 0x7fffffff9a85 thread T0
    #0 0x555555595181 in fread (/home/yty/giftrans/fuzzing/giftrans-asan+0x41181) (BuildId: 95e405168f019cdb911018f305549a9ecdd67c3d)
    #1 0x55555563cf94 in giftrans /home/yty/giftrans/giftrans.c:619:11
    #2 0x555555643790 in main /home/yty/giftrans/giftrans.c:970:10
    #3 0x7ffff7ceac89  (/lib/x86_64-linux-gnu/libc.so.6+0x27c89) (BuildId: 2e01923fea4ad9f7fa50fe24e0f3385a45a6cd1c)
    #4 0x7ffff7cead44 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x27d44) (BuildId: 2e01923fea4ad9f7fa50fe24e0f3385a45a6cd1c)
    #5 0x555555579460 in _start (/home/yty/giftrans/fuzzing/giftrans-asan+0x25460) (BuildId: 95e405168f019cdb911018f305549a9ecdd67c3d)

Address 0x7fffffff9a85 is located in stack of thread T0 at offset 1861 in frame
    #0 0x55555563b42f in giftrans /home/yty/giftrans/giftrans.c:384

  This frame has 4 object(s):
    [32, 800) 'buffer' (line 385)
    [928, 935) 'lsd' (line 385)
    [960, 1728) 'gct' (line 385)
    [1856, 1861) 'gce' (line 385) <== Memory access at offset 1861 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism, swapcontext or vfork
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow (/home/yty/giftrans/fuzzing/giftrans-asan+0x41181) (BuildId: 95e405168f019cdb911018f305549a9ecdd67c3d) in fread
Shadow bytes around the buggy address:
  0x10007fff7300: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff7310: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff7320: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff7330: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff7340: f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2
=>0x10007fff7350:[05]f3 f3 f3 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff7360: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff7370: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff7380: f1 f1 f1 f1 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff7390: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff73a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
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
==2717495==ABORTING
```