## Level 1:

A program wants a 3 digit code. Because this is just 1000 possible combinations I run a loop and grep the one that is correct:

```console
level1@io:/levels$ for a in {0..9}; do for b in {0..9}; do for c in {0..9}; do echo "$a$b$c" ; echo "$a$b$c" | ./level01 ; done ; done ; done | grep "Congrats" -B 1
```

And got the result:

```console
level1@io:/levels$ for a in {0..9}; do for b in {0..9}; do for c in {0..9}; do echo "$a$b$c" ; echo "$a$b$c" | ./level01 ; done ; done ; done | grep "Congrats" -B 1
Enter the 3 digit passcode to enter: 271
Enter the 3 digit passcode to enter: Congrats you found it, now read the password for level2 from /home/level2/.pass
level1@io:/levels$
```

Note: I used th B option for grep to get the number I pring before running the program with it as the answer is not part of the result.


### Different methods:

Using **objdump**:


```console
level1@io:/levels$ objdump -d ./level01

./level01:     file format elf32-i386


Disassembly of section .text:

08048080 <_start>:
 8048080:	68 28 91 04 08       	push   $0x8049128
 8048085:	e8 85 00 00 00       	call   804810f <puts>
 804808a:	e8 10 00 00 00       	call   804809f <fscanf>
 804808f:	3d 0f 01 00 00       	cmp    $0x10f,%eax
 8048094:	0f 84 42 00 00 00    	je     80480dc <YouWin>
 804809a:	e8 64 00 00 00       	call   8048103 <exit>
```

Looking at the "cmp" instruction we can see our input be compared to value: **0x10f** which stands for hex value of **271** which is indeed the solution.

Using **gdb**

```console
level1@io:/levels$ gdb level01
```

We can have a look at the machine instructions, after we set a breakpoint and then dump some instructions:
 
```console
(gdb) break main
Breakpoint 1 at 0x8048080
(gdb) start
Temporary breakpoint 2 at 0x8048080
Starting program: /levels/level01 

Breakpoint 1, 0x08048080 in _start ()
(gdb) x/10i $pc-1
   0x804807f:	add    %ch,0x28(%eax)
   0x8048082 <_start+2>:	xchg   %eax,%ecx
   0x8048083 <_start+3>:	add    $0x8,%al
   0x8048085 <_start+5>:	call   0x804810f
   0x804808a <_start+10>:	call   0x804809f
   0x804808f <_start+15>:	cmp    $0x10f,%eax
   0x8048094 <_start+20>:	je     0x80480dc
   0x804809a <_start+26>:	call   0x8048103
   0x804809f:	sub    $0x1000,%esp
   0x80480a5:	mov    $0x3,%eax
```

We can see the same thing as with objdump, a **cmp** instruction with a value of 0x10f which is 271.

<!-- Level 2 password => XNWFtWKWHhaaXoKI -->