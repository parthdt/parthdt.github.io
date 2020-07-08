---
layout: post
title:  OverTheWire Bandit Levels 1-10
excerpt: "First attempt at wargames, hopefully a good first writeup as well :D"
categories: overthewire wargames

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

##### Remaining levels coming soon :D

