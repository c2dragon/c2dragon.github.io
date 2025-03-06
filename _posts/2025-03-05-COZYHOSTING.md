---
title: "HTB | COZYHOSTING"
date: 2025-03-05 00:00:00 +0800
categories: [HTB]
tags: [Linux]                    
---

![img-description](/assets/img/HTB/htb.png)

## Initial Access - Exposed legitimate cookie leads to portal access and Command Injection

**Vulnerability Explanation:** An exposed legitimate cookie allows for an attacker to access the website as user 'kanderson'. The website is vulnerable to command injection via the username field. 

**Vulnerability Fix:** Remove the exposed cookie.

**Severity:** Critical

**Steps to reproduce the attack:** Christian began with an nmap port scan and discovered ports 22 and 80 open. Port 80 is hosting a web server and with further enumeration, a hidden directory '/actuator/sessions' was found. Inside the directory is a valid cookie to user 'kanderson'. This give access to the site where the 'username' field is vulnerable to command injection. A reverse shell is able to be established with command injection.  

**Port Scan Results**

**TCP:** 22, 80

**UDP:** N/A

Use nmap to scan the target for open ports.

TCP ports 22, 80 were discovered.

![img-description](/assets/img/HTB/COZYHOSTING/1.png)

Add 'cozyhosting.htb' to /etc/hosts so we can access the site.

![img-description](/assets/img/HTB/COZYHOSTING/2.png)
![img-description](/assets/img/HTB/COZYHOSTING/3.png)

Perform directory enumeration finds an interesting 'error' page. Looking further into the error page reveals the site is using something called 'Spring Boot' and 'Spring Boot' is has a vulnerable page at "/acuator/sessions".
![img-description](/assets/img/HTB/COZYHOSTING/4.png)![img-description](/assets/img/HTB/COZYHOSTING/5.png)![img-description](/assets/img/HTB/COZYHOSTING/6.png)
Navigating to the directory "/actuator/sessions" reveals a valid cookie for user 'kanderson'.

![img-description](/assets/img/HTB/COZYHOSTING/7.png)
![img-description](/assets/img/HTB/COZYHOSTING/8.png)

Replacing our cookie with the newly found one give us access to the site as user "K. Anderson."

![img-description](/assets/img/HTB/COZYHOSTING/9.png)

Under the connection setting. The 'Username' field is vulnerable to a command injection attack. Open a netcat listener and execute the reverse shell command on the system.

```shell
nc -nlvp 4444
```

![img-description](/assets/img/HTB/COZYHOSTING/10.png)
![img-description](/assets/img/HTB/COZYHOSTING/11.png)
![img-description](/assets/img/HTB/COZYHOSTING/12.png)

We now have initial access to the target machine as user 'app'.

## Privilege Escalation - Lateral movement via Database hash. Root via GTFOBins

**Vulnerability Explanation:** An exposed hash in an exposed sql database is vulnerable to a dictionary attack. The newly accessible account is vulnerable to root privilege escalation by GTFOBins.

**Vulnerability Fix:** Change password to a stronger password. Remove hashes from the database.

**Severity:** Critical

**Steps to reproduce the attack:** Christian enumerated the file system and found an interesting .jar file in the /tmp directory. This holds a password for a SQL database which in turn hold a password hash. Christian cracked the hash using hashcat and used the credentials to gain access to user 'josh' acccount. Josh is able to use the sudo command and root access can be achieved by commands in GTFOBins.

Locate interesting file in /tmp.

![img-description](/assets/img/HTB/COZYHOSTING/13.png)

Username and password for the database is found in one of the files. Username:postgres Password:Vg&nvzzAQ7XxR

![img-description](/assets/img/HTB/COZYHOSTING/14.png)
![img-description](/assets/img/HTB/COZYHOSTING/15.png)

Looking through the database reveals a hash. Copy the hash into a file.

![img-description](/assets/img/HTB/COZYHOSTING/16.png)
![img-description](/assets/img/HTB/COZYHOSTING/17.png)
![img-description](/assets/img/HTB/COZYHOSTING/18.png)

Crack the newly created hash file using hashcat. The cracked password is manchesterunited.

![img-description](/assets/img/HTB/COZYHOSTING/19.png)

Login via SSH using Josh(a user found on the system during the initial system enumeration) and the cracked password.

![img-description](/assets/img/HTB/COZYHOSTING/20.png)

User 'josh' has sudo privileges with sudo. This can be exploited using GTFOBins.

![img-description](/assets/img/HTB/COZYHOSTING/21.png)

Use the commands given by GTFOBins.

![img-description](/assets/img/HTB/COZYHOSTING/22.png)
We now have root access to the system.

## Post Exploitation

Since this is a CTF the objective is only to retrieve the flag located in the /root directory as a privileged user. 

In a real world assessment, we would attempt to add a back door for continuous access to the machine.