---
title: Blunder Walkthrough
description: Description placeholder
layout: page
---

Machine: Blunder
IP: 10.10.10.191

## Reconassiance

I started off doing a port scan `nmap -sC -sV 10.10.10.191`

![NMap results]({{ site.url }}/images/htb/blunder/nmap.jpg)

and saw that there was a website so I fired up a browser and took a look.

![Website landing page]({{ site.url }}/images/htb/blunder/website.jpg)

digging around in the source code for the blog I found a couple clues that seemed like they would be useful later.

![bludit version]({{ site.url }}/images/htb/blunder/version.jpg)

Looking for other files that might be accessable through the website, I loaded up OWASP's DIrBuster

![DirBuster]({{ site.url }}/images/htb/blunder/dirbuster.jpg)

which among other things found a login page

![DirBuster results]({{ site.url }}/images/htb/blunder/dirbuster_results.jpg)

Which gave me a new objective, finding a username/password to login with. I was initially looking at the blog for likely usernames to throw some brute force attacks at to no avail, then I realized that when I ran DirBuster I'd had the file type set to .php. I ran it again with a more general search and low and behold
![todo list]({{ site.url }}/images/htb/blunder/todo.jpg)

it found a todo.txt.

which conviently contained the username fergus. Now on to a password.

## Gaining a Foothold

Doing some research on bludit and checking its security documnetation I saw that it has some brute force mitigation that will blacklist an ip address after enough failed loging attempts. [brute-force-protection](https://docs.bludit.com/en/security/brute-force-protection) Looking up that version number from earlier, 3.9.2, I found a way to bypass the brute force mitigation at [https://rastating.github.io/bludit-brute-force-mitigation-bypass/](https://rastating.github.io/bludit-brute-force-mitigation-bypass/) which also described how it worked pretty nicely.

In order to prevent locking out everyone when say, a load balanced was fowarding requests to the login server, bludit used the `HTTP_X-FORWARDED_FOR` header to find the client's IP. What it didn't do, was validate the IP address doing the forwarding. By claiming to be forwarding the login attempt from a different fake IP Address every time, the fake ip address end up on the blacklist instead of our real IP, so the brute force attempt can keep on going.

rastating.github.io had a nice proof of concept for the rate limit bypass, but I did have to make a few modifications to the python script to make it use a wordlist for the passwords. [rate_limit_bypass.py](https://github.com/lithrion/htb_scripts/blob/main/blunder/rate_limit_bypass.py) I launched the brute force attack and....nothing. Pretty certain that this rate limit bypass was the exploit that was that was to get into this machine I tried another wordlist, and still nothing. Wracking my brain for a possible list of words where the password could be hiding, I eventually tried the content of the blog.

I used the tool cewl, which spiders a url and returns a list of the words that it finds. Running `cewl 10.10.10.191 >> wordlist.txt`

Running the rate limit bypass script again with that new wordlist did find a password, RolandDeschain. Onward!

From the bludit dashboard
![Bludit Dashboard]({{ site.url }}/images/htb/blunder/dashboard.jpg)

I was able to use CVE-2019-16113 to gain shell access. The bludit content creation page allows image files to be uploaded, but it doesn't check that the file being uploaded is really an image. This allows remove code execution, which I used to establish a reverse shell. I found the python script for the exploit at [https://github.com/cybervaca/CVE-2019-16113](https://github.com/cybervaca/CVE-2019-16113), and made a couple edits to use blunder's IP and fergus' credentials.

First I started a listener for the reverse shell 
`nc -lnvp 4444`
and then ran the script
`python cve-2019-16113.py`
which started up the reverse shell. Since the shell netcat establishes is kinda limited I upgraded it
`python -c 'import pty; pty.spawn("/bin/bash")`

Now with shell access, I still didn't have the user flag yet, so I dug around a little.

## The User flag

![ls]({{ site.url }}/images/htb/blunder/ls.jpg)
I found a newer version of bludit, 3.10.0a and inside there found a users.php.
![users.php]({{ site.url }}/images/htb/blunder/users.jpg)

Which has the username Hugo and the hash of his password. I tossed that passsword hash into crackstation.net which gave back Hugo's password, Password120.

From there I switched users to Hugo
![hugo login]({{ site.url }}/images/htb/blunder/hugo.jpg)

and checking his home directory.
![home directory]({{ site.url }}/images/htb/blunder/home.jpg)
revealed the user flag.

## Escalting to root access

Checking Hugo's privilages, I found that he had sudo access as any users except for root by checking `sudo -l`
![sudo access]({{ site.url }}/images/htb/blunder/sudo.jpg)

This actually led me to a handful of red herrings. I found a couple different system users on the machine with versions that had privilage escalation vulnerabilities associated with them, but as I dug deeper I found the vulnerabilities were also tied to other services that weren't on the machine to take advantage of. Eventually I tracked down CVE-2019-14287, a vulnerability is sudo versions prior to 1.8.28. Turns out, that sudo would treat negative user IDs as if they were 0, which is root's user ID. Since Huge can use sudo as any user other than root (ID 0), it'll let him run sudo with an ID of say, -1. Then that ID of -1 actually gets treated as ID 0, and hugo just ran a command as root.

Using the -u flag on sudo to specific a user ID, and \\#-1 to enter the negative value, I ran the command
`sudo -u \#-1 bash`
![root access]({{ site.url }}/images/htb/blunder/root.jpg)
to open up a new bash shell as root. Then I moved over to root's home directory, and there was the root flag. Mission accomplished.
![victory]({{ site.url }}/images/htb/blunder/flag.jpg)





