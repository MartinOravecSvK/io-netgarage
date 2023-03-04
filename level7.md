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

Change the return address inside the memcpy to point to the shell code or at leaste the NOP slide.