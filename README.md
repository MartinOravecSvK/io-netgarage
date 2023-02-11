# io-netgarage
IO net garage walk-through


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

Note: I used th B option for grep to get the number I pring before running the program with it as the answer is not part of the result;