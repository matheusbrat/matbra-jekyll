---
layout: post
title: Usando uma porta serial para gravar um esp
lang: pt_BR
description: Usando uma porta serial remota com ser2net para gravar um esp utilizando Raspberry pi
tags: linux ser2net socat raspberry esp32 esp8266 serial port
comments: true
--- 

Recentemente eu voltei a brincar com meu ESP8266 dado que decidi fazer minha casa mais inteligente. Olhando as placas que eu tinha disponivel notei que eu não possuia nenhuma placa pra gravar em 3.3 mas olhando mais perto notei um raspberry pi e eu poderia simplesmente usa-lo

Depois de fazer as conexões e conseguir gravar o esp utilizando o raspberry pi eu senti que era muito chato programar usando o mesmo ou vnc ou coisas parecidas. Então eu decidi tentar utilizar uma porta serial remota

Inicialmente eu tentei com socat mas não consegui fazer o mesmo funcionar, me parece que ele não encaminha alguns sinais. Depois encontrei o ser2net que é compativel com o RFC2217.

Para instalar o ser2net no raspberry eu utilizei:
```
$ sudo apt-get install ser2net
```

Posteriormente para criar um tunel e expor ele na minha maquina utilizei:
```
$ ssh -L 8086:localhost:8086 pi@PI_ADDRESS '/usr/sbin/ser2net -d -C "8086:raw:600:/dev/ttyAMA0:115200"'
``` 

Basicamente estou fazendo um tunel da minha porta 8086 para o dispositivo remoto na porta 8086 com permissões 600 na porta /dev/ttyAMA0 e baudrate de 1152000.

Para conseguir gravar o mesmo usei o esptool da seguinte forma:
```
$ esptool.py -p socket://localhost:8086 write_flash -fm dio 0x000000 BasicOTA.ino.generic.bin
```
Note que utilizei o -p socket:// para me comunicar com o socket e gravar remotamente funcionou.


Espero que isso seja util para você também,
Matheus