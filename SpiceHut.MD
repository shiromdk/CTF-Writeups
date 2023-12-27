## THM - Startup
### https://tryhackme.com/room/startup

### Introduction
We are Spice Hut, a new startup company that just made it big! We offer a variety of spices and club sandwiches (in case you get hungry), but that is not why you are here. To be truthful, we aren't sure if our developers know what they are doing and our security concerns are rising. We ask that you perform a thorough penetration test and try to own root. Good luck!

The goal is to find the secret spicy soup recipe and to get the contents of user.txt and root.txt

### First steps
The first step was to check the website itself to see if we could glean anything there. On initial inspection it seems to just be a plain html page with no js of any kind

Next we run nmap to check what ports are open. We get this result
```
root@kali:~# sudo nmap -sV -sC -T4 10.10.0.140
Starting Nmap 7.92 ( https://nmap.org ) at 2023-12-27 04:46 EST
Nmap scan report for 10.10.0.140
Host is up (0.31s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.4.51.140
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
|   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
|_  256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Maintenance
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
From this we see that ports 21,22 and 80 are open.
The website is running on a Apache Server but
most importantly we can login to the ftp server anonymously.

We used gobuster to find what ever directories might be there. 
```
root@kali:~/Spice# gobuster dir -w ../SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -u 10.10.0.140
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.0.140
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                ../SecLists-master/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/files                (Status: 301) [Size: 310] [--> http://10.10.0.140/files/]
/server-status        (Status: 403) [Size: 276]
```

### Reverse Shell
The next step is to login via ftp and upload this reverse shell https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php. The ftp folder seems like a good place to put it.

We listen on our attacking system using nc and then to run the php script we can just go to http://WEBSITE_URL/files/ftp/php-reverse-shell.php


### Gaining access

Once we are in the first thing I do is stabilise the shell with this
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
With this user account we can immediately get the answer to first question by outputting the contents of recipe.txt

We also see an an incidents folder which is unusual. Inside we find a suspicious.pcapng. We can copy that to the ftp folder to make it easier for us to download and inspect it.

Inspecting it we find on stream 7 sudo -l attempts with this password
```
c4ntg3t3n0ughsp1c3
```
We try this with the lennie account and it works. We can immediately read the user.txt file and we can see inside the /home/lennie directory there is a Documents folder and a scripts folder


If we inspect the scripts we see a planner.sh and a startup_list.txt
The planner.sh appears to add whatever the list variable is to the txt file. and then runs /etc/print.sh

It looks like we have write access to /etc/print.sh so lets see what we can do with this.