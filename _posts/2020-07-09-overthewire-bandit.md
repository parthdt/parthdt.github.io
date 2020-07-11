---
layout: post
title:  OverTheWire Bandit Write-Up
excerpt: "First attempt at wargames, hopefully a good first write-up as well :D"
categories: overthewire wargames write-up

---

## Level 0

Level one was simple SSHing into the server using:

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

Since every level had a different password, I saved each level's password into a filename called ```bandit[NUMBER]``` in the ```passwords``` directory and created a simple script to automate the SSH process:

```bash
NUMBER=${1?Error: no file-number given}
sshpass -f bandit$NUMBER ssh passwords/bandit$NUMBER@bandit.labs.overthewire.org -p 2220
```

``` sshpass ``` let's you SSH and provide the password all at once.

Each time I had to connect to a level, all I had to do was:

```bash
./autoSSH.sh <level-number>
```

``` autoSSH.sh ``` is the name of the script.

## Level1

The password was in a file named ``` - ```. Now ``` cat - ``` will just return whatever is inputted as it is stdin; so we should specify the path of the file: ``` cat ./- ``` will give the password.

## Level 2

The password was in a file that had spaces in its name. Either ``` cat spaces\ in\ this\ filename ``` or ``` cat "spaces in this filename" ``` should do the trick.

## Level 3

The password was in a hidden file in the ``` inhere ``` directory. For the password: 
```
cat inhere/.hidden
```

## Level 4

File is in the only "human-readable" file in the ``` inhere ``` directory.

Running ``` file inhere/* ``` gives us:
```
inhere/-file00: data
inhere/-file01: data
inhere/-file02: data
inhere/-file03: data
inhere/-file04: data
inhere/-file05: data
inhere/-file06: data
inhere/-file07: ASCII text
inhere/-file08: data
inhere/-file09: data
```

``` -file07 ``` is the only ASCII (human-readable) file.
    
``` cat inhere/-file07 ``` for the password.



## Level 5

We have to find a human-readable file with a particular file size (1033 bytes) from a gajillion (maybe not) files. Let's turn to the ```find``` command along with ```du``` for the file size:

```find -type f -exec du -b {} \; | grep 1033``` finds the sizes of all files in bytes, then greps 1033 to get the required file.
   Output: 
   ```
   1033    ./maybehere07/.file2
   ```
  
``` cat ./maybehere07/.file2 ``` for the PW.

## Level 6

Finding a file anywhere on the server, owned by user bandit7 and group bandit6, 33 bytes in size.

Running this command : ```find / -type f -user bandit7 -group bandit6 -exec du -b {} \;``` should find the file. It finds all files anywhere in the server (```/``` directory) owned by the appropriate people. But it gives a lot of permission denied errors:
```
find: ‘/root’: Permission denied
find: ‘/home/bandit28-git’: Permission denied
find: ‘/home/bandit30-git’: Permission denied
find: ‘/home/bandit5/inhere’: Permission denied
find: ‘/home/bandit27-git’: Permission denied
find: ‘/home/bandit29-git’: Permission denied
find: ‘/home/bandit31-git’: Permission denied
find: ‘/lost+found’: Permission denied
.
.
.
```

Run ```find / -type f -user bandit7 -group bandit6 -exec du -b {} \; 2>/dev/null``` this time. ```2>/dev/null``` redirects ```stderr (2)```, to ```/dev/null```; meaning now you only see output without error messages:     
```
33      /var/lib/dpkg/info/bandit7.password
```

```cat /var/lib/dpkg/info/bandit7.password``` for the PW.

## Level 7

This one was a simple cat and grep : ```cat data.txt | grep millionth```

## Level 8

We had to find the unique line here, one that occurs only once.

This command does it: ```sort data.txt | uniq -u```. It first sorts all the lines, then compares adjacent lines and shows only those that occur once (unique).

## Level 9

The password is in one of the human readable strings in ```data.txt``` preceded by some ```=```'s. ```strings data.txt | grep ===``` gives us:
```
========== the*2i"4
========== password
Z)========== is
&========== truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk
```

Very clearly, the password is ```truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk```. 

## Level 10

The password is in data.txt, but it is base64'd! Damn hard to decode, but we do it with the inbuilt base64 utility: 

```cat data.txt | base64 -d``` reveals : ```The password is IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR```. Sweet!

## Level 11

The password was in ```data.txt``` and rotated by 13 positions (ROT13). The rot13 command wasn't there in the bandit server, so copied the contents of the file and did the following in my machine:
```bash
echo "Gur cnffjbeq vf 5Gr8L4qetPEsPk8htqjhRK8XSP6x2RHh" | rot13
The password is 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu
```

The intended solution (I think) was however:
```bash
 cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

This tr command basically "transaltes" each char to (char+13), same as rot13. Sweet :D

## Level 12

This one had a file ```data.txt``` which contained the hexdump of another file. Using ```xxd -r data.txt``` we can get the reversed binary. This binary was compressed with either ```gzip```, ```bzip2``` or was a ```tar``` file. Moved the file to a folder in /tmp and used file command on each step, then applied the appropriate decompression command based on the compression algorithm.
```
gzip -d <file> for gzip compressed files
bzip2 -d <file for bzip2 compressed files
tar -xvf <file> for the posix tar files
```

## Level 13

In this level we had an SSH private key for the bandit14 user, so simply execute:
```bash
ssh -i sshkey.private bandit14@localhost
```

For the password: ```cat /etc/bandit-pass/bandit14```

## Level 14

To get next level's password, submit this level's password to localhost, port 30000. We can use ```nc``` for this:
```bash
cat /etc/bandit_pass/bandit14 | nc localhost 30000
```

## Level 15

We had to submit the password to localhost port 30001 via ssl connection. We can use ```openssl s_client``` for this. I did this first:
```bash
cat /etc/bandit_pass/bandit15 | openssl s_client -connect localhost:3000```
 but couldn't get the password. Then I saw the prompt to add the ```ign_eof``` tag. Added it and got the password:
```bash
cat /etc/bandit_pass/bandit15 | openssl s_client -ign_eof -connect localhost:30001
.
.
.
Correct!
cluFn7wTiGryunymYOu4RcffSxQluehd
```

The ```ign_eof``` tag tells the server to not close file immediately after eof (we press enter after the command).

The best way however was:
```bash
cat /etc/bandit_pass/bandit15 | openssl s_client -quiet -connect localhost:30001
```

The ```quiet``` flag turns on the previous flag, and doesn't print excess session and certificate info, giving us a prettier output:
```bash
cat /etc/bandit_pass/bandit15 | openssl s_client -quiet -connect localhost:30001

depth=0 CN = localhost
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = localhost
verify return:1
Correct!
cluFn7wTiGryunymYOu4RcffSxQluehd
```
## Level 16

For this level we had to perform a port scan, one of the open ports using ssl gave back the creds for the next level, upon receiving the current password.

Option 1, Nmap:
```bash
nmap -p 31000-32000 localhost

Starting Nmap 7.40 ( https://nmap.org ) at 2020-07-09 20:10 CEST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00032s latency).
Not shown: 996 closed ports
PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds
```

Option 2, netcat inbuilt port scan utility:
```bash
nc -zv localhost 31000-32000
localhost [127.0.0.1] 31960 (?) open
localhost [127.0.0.1] 31790 (?) open
localhost [127.0.0.1] 31691 (?) open
localhost [127.0.0.1] 31518 (?) open
localhost [127.0.0.1] 31046 (?) open
```

One of these ports gave back a private SSH key for the next level.

## Level 17

The password is the only line changed in ```passwords.new```, rest is same as ```passwords.old```. Ran ```diff``` command against the two files and got the pw.

## Level 18

The password for this level is in the ```readme``` file. But, we get logged out immediately as we ssh into the server. We can enter a command at the end of the sshpass command to execute as we enter in, so this is how I solved it (from my machine):
```bash
sshpass -f passwords/bandit18 ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```

## Level 19

We have to use the setuid binary present in the home directory for this one:
```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

## Level 20

For this level, the setuid binary listened to any port we mention, and returns the next pw, if the current pw is sent. So, on one shell:
```bash
 ./suconnect 6969
Read: GbKksEFF4yrVs6il55v6gwY5aVje5f0j
Password matches, sending next password
```

We get this message as we sent the correct password from another shell:
```bash
nc -lvp 6969
listening on [any] 6969 ...
connect to [127.0.0.1] from localhost [127.0.0.1] 49888
GbKksEFF4yrVs6il55v6gwY5aVje5f0j
gE269g2h3mw3pwgrj0Ha9Uoqen1c9DGr
```

Basically listening on the next level for a pw from the current level.

Feeling like a pentester now :smirk:

## Level 21

This level had the following cronjob (a job performed at regular intervals of time) :
```bash
cat /usr/bin/cronjob_bandit22.sh


#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

Reading the ```/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv``` file gave the pw for the next level.

## Level 22

This level had the following cronjob:
```bash
cat /usr/bin/cronjob_bandit23.sh

#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```
 It wrote the password to the ```/tmp/$mytarget``` directory. So I changed the myname to bandit23, then read the correct file:
```bash
bandit22@bandit:/etc/cron.d$ mytarget=$(echo I am user bandit23 | md5sum | cut -d ' ' -f 1)
bandit22@bandit:/etc/cron.d$ cat /tmp/$mytarget
jc1udXuA1tiHqjIsL8yaapX5XIAI6i0nash
```

## Level 23

We see the following cronjob in this level:
```bash

#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname
echo "Executing and deleting all scripts in /var/spool/$myname:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" ./$i)"
        if [ "${owner}" = "bandit23" ]; then
            timeout -s 9 60 ./$i
        fi
        rm -f ./$i
    fi
done
```

So, a script, if owned by us (```bandit23```) woudl get executed every minute if it is in the ```/var/spool/bandit24``` directory. Made an empty password file in the ```/tmp/myfolder``` dir, and the following script, gave it executable permissions, and copied it to the ```/var/spool/bandit24/``` and got the password a minute later :smile:

```bash
#!/bin/bash

cat /etc/bandit_pass/bandit24 > /tmp/myfolder/password
```

## Level 24

This level involved bruteforcing a 4 digit pin. Send the correct password along with the pin to port 30002 and you get the next password. My initial method was to check for each pin by sending it from the script, but that was slow for some reason. Then, I saw [this](https://www.jonyschats.nl/writeups/bandit-level-24-to-25/) write-up. So I followed the same method, wrote all the 10k combinations to a file ```passlist.txt``` then:
```bash
cat passlist.txt | nc localhost 30002
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
Wrong! Please enter the correct pincode. Try again.
Wrong! Please enter the correct pincode. Try again.
.
.
.
Wrong! Please enter the correct pincode. Try again.
Correct!
The password of user bandit25 is uNG9O58gUE7snukf3bvZ0rxhtnjzSGzG
```

##### Nice and Easy, didn't get the correct pin, don't care though.

## Level 25

We had an SSH key for the next level, but the shell closed everytime I SShed in. Looking the the ```/etc/passwd``` file, I found that it was this shell:
```bash
#!/bin/sh

export TERM=linux

more ~/text.txt
exit 0
```

Interesting.

Further reading showed me we can use commands in ```more```. So I made my terminal very small, to make ```more``` buffer the text then tried reading the password. Couldn't.

You can also open vim in ```more```, by typing ```v```. Did that, read the password, got it. 

## Level 26

This continued the previous level. Set the shell variable as ```/bin/bash``` in vim, then run it. You will find a setuid binary similar to one of the previous levels, letting you execute any command as ```bandit27```. Use this setuid binary to read the password.

## Level 27

We just had to clone a git repo: ```ssh://bandit27-git@localhost/home/bandit27-git/repo```, and the password was in the README file inside.

## Level 28

Similar to the previous level, we had to clone a git repo. But this time the README had the following contents:
```
# Bandit Notes
Some notes for level29 of bandit.

## credentials

- username: bandit29
- password: xxxxxxxxxx
```

Checked the logs (```git log```) to find initial commits, one fixed the credentials apparently. Use ```git show``` and you'll see the password.

##### Hack the Planet! :smile:

## Level 29

Same, git cloning. This time the contents of the README read:
```
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit30
- password: <no passwords in production!>
```

This indicated that there may be more branches (production, and.... development?!). See all branches by running ```git branch -a```. You will see a ```dev``` branch. Then do ```git checkout dev```, the password will be in the README.

## Level 30

In this level, after cloning the repo, the README file just says:
```
just an epmty file... muahaha
```

After some digging I found a tag: ```secret```. Used ```git show --all``` to find the password.

## Level 31

We had to add a ```key.txt``` file, then commit the changes. But, the gitignore file ignored ```.txt``` files. Remove that, then commit the file, push the changes and the password is retrieved.

## Level 32

We are trapped inside an uppercase shell in this one. After some reading around the net, I found a cool trick. Since the uppercase shell is still ```/bin/sh```, just converting everything to uppercase, you can run ```$0```, to get ```/bin/sh```, since that was the first command run (```$0```). Then just cat the password.

##### And, that's that! Really enjoyed the wargame, learnt a lot. Looking forward to the next wargame.:relieved:
{: .notice}


