---
layout: post
title: Impressora conectada ao Raspberry PI acessivel da sua rede local.
lang: pt_BR
description: Impressora conectada ao Raspberry PI acessivel da sua rede local.
tags: linux raspberry pi network printer cups hplip
comments: true
--- 

Boa noite,

Por um longo periodo meu pai tem reclamado que usar a impressora não é pratico o bastante, para resolver isso decidi instalar um Raspberry PI Zero W conectado a impressora e compartilhar a mesma usando CUPS.

Inicialmente você deve conectar ao RPi e instalar CUPS.

```
sudo apt-get install cups
```

Se você deseja ter uma interface web para configurar CUPS do seu computador, atualize `/etc/cups/cupsd.conf`

```
sudo vim /etc/cups/cupsd.conf
```

Encontre a linha:
```
Listen localhost:631
```

Atualize a mesma para:
```
# Listen localhost:631
Port 631
```

Você terá multiplas linhas `<Location`, se você deseja ser capaz de configurar somente do seu computador, adicione `Allow from SEU_IP` para todas as secoes. Exemplo:
```
<Location />
  Order allow,deny
  Allow from 10.0.0.2
</Location>
```
(Se você deseja poder fazer de qualquer computador, use Allow from all)


Adicione o seu usuario  your (no meu caso pi) ao grupo `lpadmin`
```
sudo usermod -a -G lpadmin pi
```

Acesse o ip do Raspberry PI no seu navegador na porte 631 (https://RPI_IP:631/).

Acesse `Administration - Add printer`. Você deve ver sua impressora em "Local printers", adicione a mesma lembrando de marcar a checkbox para compartilhar.

No caso de você usar uma impressora HP e a mesma não estiver funcionando, tente instalar o hplip.

```
sudo apt-get install hplip
```

Reinicie e tente acessar a pagina novamente.


See you,
Matheus