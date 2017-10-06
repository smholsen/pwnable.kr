# 1 - fd
The first challenge from pwnable.kr

## Description
Mommy! what is a file descriptor in Linux?

* try to play the wargame your self but if you are ABSOLUTE beginner, follow this tutorial link: https://www.youtube.com/watch?v=blAxTfcW9VU

ssh fd@pwnable.kr -p2222 (pw:guest)

## Solution

After ssh'ing into the vm we can list the resources and permissions we have to work with.

`ls -l`


```
-r-sr-x--- 1 fd_pwn fd   7322 Jun 11  2014 fd
-rw-r--r-- 1 root   root  418 Jun 11  2014 fd.c
-r--r----- 1 fd_pwn root   50 Jun 11  2014 flag
```

We can see that we have 3 files. Running `whoami` shows us that we are logged in as the user fd. Running `groups` shows us that that we belong to 1 group named fd.
Thus we can see from the permissions list that we have both read and execute permissions for the fd executable file, and read permissions for the fd.c file.

Lets take a look at fd.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
        if(argc<2){
                printf("pass argv[1] a number\n");
                return 0;
        }
        int fd = atoi( argv[1] ) - 0x1234;
        int len = 0;
        len = read(fd, buf, 32);
        if(!strcmp("LETMEWIN\n", buf)){
                printf("good job :)\n");
                system("/bin/cat flag");
                exit(0);
        }
        printf("learn about Linux file IO\n");
        return 0;

}

```

The first if check verifies using the argument counter that the argument vector contains more than 2 elements. Thus checking if we have passed anything as input. Further, the program uses atoi() which turns our input into an int, subtracts 0x1234 from it, and stores it in our fd variable.
Next the program reads from our filedescriptor, defined by our fd variable, into the 32 byte char buffer, and limits it by 32 byte. (In turn not allowing for any overflow attacks, as it corresponds to the allocated size).
We can then lookup the string comparator function strcmp, and see that it will return 0 if the compared strings are equal. (resolving to false). If we can use this to enter the if statement, we see that the program will cat the content of the flag using the command processor.
This is good news for us, as we now know that all we need to do is to make the read function use a file descriptor where we can input a string that will let us enter the if statement.

If we look up Linux File Descriptors we can find that the integer value 0 will resolve to the *Standard input, (stdin)*.

Ok, so if we can ensure that fd == 0, and provide the string "LETMEWIN", (the newline is appended), to stdin, we will hopefully be able to read the flag.
We know that whatever integer we provide as an argument to the program will be subtracted by hex 1234 before the read function is executed, so we will need to give the exact same value in, (as integer). 0x1234 corresponds to an integer value of 4660. Then we simply pipe the string to the program.

### Lets try

```
fd@ubuntu:~$ echo "LETMEWIN" | ./fd 4660
good job :)
mommy! I think I know what a file descriptor is!!

```
