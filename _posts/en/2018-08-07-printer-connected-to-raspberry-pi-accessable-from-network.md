---
layout: post
title: Printer connected to Raspberry PI accessable from network.
lang: en
description: Printer connected to Raspberry PI accessable from network.
tags: linux raspberry pi network printer cups hplip
comments: true
--- 

Hey guys,

For a long time my father has beem complaining that using the printer wasn't practical enough, so to solve this I decided to add a Raspberry pi Zero W connected to my printer (HP Deskjet F2050) and share the printer using CUPS.

Initially you need to connect to your RPi and install CUPS.

```
sudo apt-get install cups
```

If you want to have a webinterface to configure it from your local network, update `/etc/cups/cupsd.conf`

```
sudo vim /etc/cups/cupsd.conf
```

Find the line:
```
Listen localhost:631
```

And update it to:
```
# Listen localhost:631
Port 631
```

You will have multiple `<Location`, if you want to be able to access only from your computer, add `Allow from YOUR_IP` for every section. Example:
```
<Location />
  Order allow,deny
  Allow from 10.0.0.2
</Location>
```
(If you want from any, use Allow from all)

Add your user (in my case PI) to `lpadmin` group.
```
sudo usermod -a -G lpadmin pi
```

Access your Raspberry Pi ip on your browser on port 631 (https://RPI_IP:631/).

Go to `Administration - Add printer` Menu. You should see your local printer there, select it and follow the wizard to setup it. 

If you're using HP printer and can't find yours, try:

```
sudo apt-get install hplip
```

And reboot.

Let me know if you have any problems.

See you,
Matheus