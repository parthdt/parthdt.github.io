---
layout: post
title:  TryHackMe Bounty Hunter Writeup
excerpt: "Write-up for THM Bounty Hunter"
categories: tryhackme write-up

---

## THM - Bounty Hunter


### Initial thoughts, Enumeration

Easy difficulty challenge.

We are given an IP address, and the task question points towards enumeration and finding the user and root flags. Doing a default nmap scans we see the ports : `21,22,80` open. So, FTP, SSH and HTTP.

### Initial questions

Firstly, we login to the ftp server using the anonymous user. We see there are two files in the server. One file, the `tasks.txt` contains the a note left by a user, `lin`.   

We also find a file `locks.txt` which seems to contain password combinations. Using a simple bash one liner, I bruteforced the ssh service using the `sshpass` command to specify the password inside the one liner itself. We find a correct password for the user `lin`. `user.txt` is in the directory we log in to.

### Root

The first command I use in such challenges is `sudo -l`. It lists binaries that the current user can run as `root`. We find `lin` can run `tar` as `root` using `sudo`. Finding the gtfobins page for `tar` reveals the command for getting a shell.    

This successfully gives root access. `root.txt` is in the `root` folder.   

<b>Done and Dusted!</b>

![easypeasy](https://media.tenor.com/JytwfmX8t3AAAAAC/spongebob-squarepants-spongebob.gif)

