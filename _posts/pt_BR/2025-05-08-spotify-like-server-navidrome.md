---
layout: post
title: Servidor como o Spotify usando navidrome e accessando via cloudflared
lang: pt_BR
description: Servidor como o Spotify usando navidrome e accessando via cloudflared
tags: navidrome spotify cloudflare
comments: true
---

Olá,

Recentemente depois de visitar alguns amigos eles compartilharam que estavam fazendo seu servidor doméstico e que estavam usando plex e navidrome. Achei bem legal, poder ter meu próprio servidor "estilo Spotify" em casa. Eu tenho algumas playlists em m3u e achei que seria legal poder escuta-las em casa ou externamente. Considerando todo o buzz ao redor de AI, decidi brincar com AI e fazer o setup.

Comecei perguntando sobre diferenças entre navidrome e plex e decidi ir com Plex, para meu app de celular decidi usar Flo (ios) e também usar cloudflare (cloudclared, tunnel) também.

Por interagir com Gemini e ChatGPT, cheguei ao seguinte:

{% highlight bash %}
services:
navidrome:
image: deluan/navidrome:latest
container_name: navidrome
user: 1000:1000
network_mode: host
ports:
- 4533:4533
volumes:
- /home/server/system/navidrome/:/data:Z
- /mnt/data/music:/music:ro
restart: unless-stopped
environment:
- TZ=America/Sao_Paulo
- SCAN_INTERVAL=1h
dns:
- 8.8.8.8
- 8.8.4.4
{% endhighlight %}

Enfrentei alguns desafios:

SELinux não estava jogando legal com volumes e algumas outras coisas, no final das contas mudei para ser permissivo
Regras de Firewall (tive que adicionar a porta do firewall também)
Tive que adicionar o DNS ali (não tenho certeza porque)
Isso tornou o serviço disponível na minha rede local. Depois disso decidi expor via cloudflare tunnel, por interagir com Gemini, cheguei ao seguinte:

{% highlight bash %}
cloudflared:
image: cloudflare/cloudflared:latest
container_name: cloudflared
restart: unless-stopped
environment:
- TUNNEL_TOKEN=&lt;cloudflaredtunneltoken>
command: tunnel run
network_mode: host
{% endhighlight %}

Acessei a web UI do cloudflare, criei um tunnel, adicionei em um domínio como navi.home.com apontando para meu IP de casa:4533 e funcionou perfeitamente. Depois tive que configurar meu app Flo para usar meu servidor de casa também, mas bem. Isso foi bem simples
