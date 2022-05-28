---
layout: post
title: CyberHeroes - TryHackMe Write-up
---

Write-up for CyberHeroes TryHackMe CTF
Hi hackers, today I’m gonna write-up CyberHeroes, a medium TryHackMe CTF.

## Flag
First, I did port scanning with rustscan and nmap to find any open ports
```
$ rustscan -a cyberheroes.thm -t 2000
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Nmap? More like slowmap.🐢

[~] The config file is expected to be at "/home/ubuntu/.rustscan.toml"
Open 10.10.40.61:22
Open 10.10.40.61:80

$ nmap -p22,80 -A -oN port.txt -T4 cyberheroes.thm
Starting Nmap 7.80 ( https://nmap.org ) at 2022-05-28 20:36 CEST
Nmap scan report for cyberheroes.thm (10.10.40.61)
Host is up (0.32s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.48 ((Ubuntu))
|_http-server-header: Apache/2.4.48 (Ubuntu)
|_http-title: CyberHeros : Index
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.72 seconds
```

We have two open ports: 22 (SSH) and 80 (HTTP) so I started testing HTTP.

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_cyberheroes/imagine_sito.png)

HTTP is hosting a web site called CyberHeroes, I immediately noticed the login page

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_cyberheroes/login.png)

That login use a javascript authentication, here's its source code:

```
function authenticate() {
      a = document.getElementById('uname')
      b = document.getElementById('pass')
      const RevereString = str => [...str].reverse().join('');
      if (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS")) { 
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function() {
          if (this.readyState == 4 && this.status == 200) {
            document.getElementById("flag").innerHTML = this.responseText ;
            document.getElementById("todel").innerHTML = "";
            document.getElementById("rm").remove() ;
          }
        };
        xhttp.open("GET", "RandomLo0o0o0o0o0o0o0o0o0o0gpath12345_Flag_"+a.value+"_"+b.value+".txt", true);
        xhttp.send();
      }
      else {
        alert("Incorrect Password, try again.. you got this hacker !")
      }
    }
```

That script checks if the username is equal to h3ck3rBoi and if the username is equal to RevereString("54321@terceSrepuS"), so the username is h3ckerBoi whereas to get password's value we can
execute the piece of script which reverse the string by using node, here's is result:
```
$ node
Welcome to Node.js v12.22.5.
Type ".help" for more information.
> const RevereString = str => [...str].reverse().join('');
undefined
> RevereString("54321@terceSrepuS")
'[REDACTED]'
> 
```
Now we have username and password, let's login

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_cyberheroes/post.jpg)

here's our flag! See you at the next post, keep hacking
