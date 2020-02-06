---
layout: post
title: Use a remote serial port to flash an esp
lang: en
description: Using a remote serial port with ser2net to flash an esp using raspberry pi
tags: linux ser2net socat raspberry esp32 esp8266 serial port
comments: true
--- 

Recently I got back to playing with some ESP8266 as I decided to make my home smarter. Taking a look on my boards I noticed I didn't have any 3.3V board to flash it. Taking a closer look I found I had a raspberry pi around, so I could simply use it. 

After doing the setup and being able to flash using the raspberry pi it felt too hard to be programming on it and using vnc or something like this. Therefore I decided to try to use a remote serial port.

At first I got to socat but I couldn't get it to work as it seems it doesn't forward some specific signals. After some googling I found ser2net which seems to be compliant with RFC2217.

To install ser2net on my raspberry pi I used:
```
$ sudo apt-get install ser2net
```

After this to create a tunnel and expose it on my machine I used:
```
$ ssh -L 8086:localhost:8086 pi@PI_ADDRESS '/usr/sbin/ser2net -d -C "8086:raw:600:/dev/ttyAMA0:115200"'
``` 

Basically I'm forwarding my local port 8086 and on the remote device on 8086, being raw with permission 600 with port /dev/ttyAMA0 and baudrate of 115200. 

To be able to flash my ESP8266 I used:
```
$ esptool.py -p socket://localhost:8086 write_flash -fm dio 0x000000 BasicOTA.ino.generic.bin
```
Note the -p socket:// with this it will use the socket to communicate.


I hope this will be helpful for you.
Matheus