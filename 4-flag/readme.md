# 4 - flag
The first challenge from pwnable.kr

## Description
Papa brought me a packed present! let's open it.

Download : http://pwnable.kr/bin/flag

This is reversing task. all you need is binary

## Solution
Ok, so there is no interaction with the server this time. Lets download the binary and see if we can find any clues.
```
$ wget http://pwnable.kr/bin/flag
$ strings flag
```
Ok so there are a ton of strings here, but the last two gives us a nice clue.
```
UPX!
UPX!
```
UPX is an open source executable packer, so we can assume the binary is packed using it. I went ahead and downloaded the program.
```
$ sudo apt-get install upx
$ upx
Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2013
UPX 3.91        Markus Oberhumer, Laszlo Molnar & John Reiser   Sep 30th 2013

Usage: upx [-123456789dlthVL] [-qvfk] [-o file] file..

Commands:
  -1     compress faster                   -9    compress better
  -d     decompress                        -l    list compressed file
  -t     test compressed file              -V    display version number
  -h     give more help                    -L    display software license
Options:
  -q     be quiet                          -v    be verbose
  -oFILE write output to 'FILE'
  -f     force compression of suspicious files
  -k     keep backup files
file..   executables to (de)compress

Type 'upx --help' for more detailed help.

UPX comes with ABSOLUTELY NO WARRANTY; for details visit http://upx.sf.net
```

Let's try and decompress the binary and debug it with gdb.

```
$ upx -d flag -o decompressed
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2013
UPX 3.91        Markus Oberhumer, Laszlo Molnar & John Reiser   Sep 30th 2013

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    887219 <-    335288   37.79%  linux/ElfAMD   decompressed

Unpacked 1 file.

```

ʘ‿ʘ

Ok, so on the next part I guess I was very lucky, as I just assumed there was a function called main.

```
$ gdb decompressed
(gdb) disas main
Dump of assembler code for function main:
   0x0000000000401164 <+0>:	push   %rbp
   0x0000000000401165 <+1>:	mov    %rsp,%rbp
   0x0000000000401168 <+4>:	sub    $0x10,%rsp
   0x000000000040116c <+8>:	mov    $0x496658,%edi
   0x0000000000401171 <+13>:	callq  0x402080 <puts>
   0x0000000000401176 <+18>:	mov    $0x64,%edi
   0x000000000040117b <+23>:	callq  0x4099d0 <malloc>
   0x0000000000401180 <+28>:	mov    %rax,-0x8(%rbp)
   0x0000000000401184 <+32>:	mov    0x2c0ee5(%rip),%rdx        # 0x6c2070 <flag>
   0x000000000040118b <+39>:	mov    -0x8(%rbp),%rax
   0x000000000040118f <+43>:	mov    %rdx,%rsi
   0x0000000000401192 <+46>:	mov    %rax,%rdi
   0x0000000000401195 <+49>:	callq  0x400320
   0x000000000040119a <+54>:	mov    $0x0,%eax
   0x000000000040119f <+59>:	leaveq
   0x00000000004011a0 <+60>:	retq   
End of assembler dump.
```
Hmm, look at that comment about a flag @ 0x6c2070
Let's try to print out whatever that points to.

 ```
 (gdb) x/s *0x6c2070
0x496628:	"UPX...? sounds like a delivery service :)"
```

｡^‿^｡
