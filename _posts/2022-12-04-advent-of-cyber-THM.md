---
layout: post
title: Advent of Cyber TryHackMe 2022
excerpt: "A walkthrough/attempt writeup for Advent of Cyber"
categories: write-up
---

> This is a writeup/ thoughts post for the Advent of Cyber 2022 by TryHackMe.

## Task 1-4: The introductory stuff

* Task 1:   
Introduction to the challenge. Interesting thing is that there are multiple giveaways of gifts; one for the whole challenge and one for each day you solve. Nice THM!
* Task 2: Rules and tutorials.
* Task 3: Socials.
* Task 4: Sales pitch.

## Task 5: The Story

There's a comic to start off, nice! :smile:   
So, the SOC has been physically breached and their website has been defaced. A puzzle has been left by the attacker(s). Let's see what the challenges are.

## Task 6, Day 1: Frameworks

Task has some materials regarding security frameworks. A unified kill chain is a unification of the MITRE ATT&CK and Cyber Kill Chain frameworks.

After reading through the material of security frameworks and the 18 phases and 3 cycles of the Unified Kill Chain, we are led to the gift shop website that has been defaced. It has 3 puzzles, corresponding to the 3 cycles of the kill chain. We have to fill each step(phase) out. After doing so (which was terribly easy), we get the flag.

## Task 7, Day 2: Log Analysis

The day of logs. A lot of material related to logging was written in today's task introduction.
Then, we have to SSH in to the server affected and analyze log files.   
We find two log files: `SSHD.log` and `webserver.log`.   

We had to use `grep` to answer simple questions like which IP address was used etc. Was a simple log file like in apache servers, so this day was straightforward and had nothing noteworthy.

## Task 8, Day 3: OSINT

Task for OSINT'ing!   
<img src='https://openseauserdata.com/files/971ad41de517bd16a620c0879b47bd13.jpg' height=200 width=200>   

The site `santagift.shop` has been defaced and is the target. So, we have to use the WHOIS database and other tools and websites (Github too) to answer certain questions.

Pretty straightforward. We find the source code on Github by searching `santagift`. The repository contains a `config.php` file, containing the answers to all the questions in this task.   

Pretty Easy till now.









