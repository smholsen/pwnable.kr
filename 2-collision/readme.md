# 2 - collision
The second challenge hosted on pwnable.kr

## Description
Daddy told me about cool MD5 hash collision today.
I wanna do something like that too!

ssh col@pwnable.kr -p2222 (pw:guest)

## Solution
Listing our resources and permissions shows us 3 files.
```
col@ubuntu:~$ ls -l
total 16
-r-sr-x--- 1 col_pwn col     7341 Jun 11  2014 col
-rw-r--r-- 1 root    root     555 Jun 12  2014 col.c
-r--r----- 1 col_pwn col_pwn   52 Jun 11  2014 flag
```

We belong to the group *col*, thus we have read access to the col.c file, and execute permissions to the executable col file.

Lets inspect the source code.
```
col@ubuntu:~$ cat col.c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```
We see that in order to print the flag we need to enter the if statement checking whether hashcode == check_password( argv[1] ).
Inspecting the check_password function we can see that the input is read in an iterative manner, over 5 iterations. We know that an int is 4 bytes, while a char is 1 byte.
Lets try to just split it into 5 evenly sized chunks.
0x21DD09EC/5 = 0x06C5CEC9 (with 4 in remainder.. blah ok that wont work..)
Ok, so my first thought now was to rather try dividing by something smaller and just filling in the remainding bytes with \x00, but our passcode is checked using strlen, which will not count null values (\x00 resolves to null).
y next attempt was to just subtraxt 0x01010101 from the original hex and divide the remainder by 4, this left a 3 in remainder still so I came up with this solution.

```
0x21DD09EC - 3 = 0x21DD09E9
0x21DD09E9 - 0x01010101 = 0x20DC08E8
0x20DC08E8 / 4 = 0x837023A
```
Great! So we can just input 0x837023A 4 times and then append 0x01010104 (0x01010101 + 3)

Ok, so now we have to figure out how to format our input. We can check if our system is big or little endian using python as such
```
col@ubuntu:~$ python -c "import sys;print(0 if sys.byteorder=='big' else 1)"
1
```
Right, so we are on a little endian system.
With this information in mind we can assume that the input we would need to provide the program would be *\x3a\x02\x37\08* 4 times, and *\x04\x01\x01\x01* 1 time.
## Lets try
```
col@ubuntu:~$ ./col `python -c 'print "\x3a\x02\x37\x08"*4+"\x04\x01\x01\x01"'`
daddy! I just managed to create a hash collision :)
```

### Awesome! ʕ•ᴥ•ʔ
