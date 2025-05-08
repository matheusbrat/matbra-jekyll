---
layout: post
title: Spotify like server using navidrome and expose via cloudflared
lang: en
description: Using navidrome to have a home server exposed with cloudflared
tags: navidrome spotify cloudflare
comments: true
---

Hello,

Recently after visiting some friends they shared they were doing their home server and that they were using plex and navidrome. I thought it was quite cool, to be able to have my own "Spotify like" server at home. I do have some playlists on m3u and I felt it would be nice to be able to listen to them at home or externally. Considering all the buzz around AI, I decided to play with AI and do the setup. 

I started asking about differences between navidrome and plex and decided to go with Plex, for my mobile phone app I decided to use Flo (ios) and also to use cloudflare (cloudclared, tunnel) as well. 

By interacting with Gemini and ChatGPT, I got to the following:

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

I faced some challenges:
- SELinux wasn't playing nice with volumes and some other stuff, in the end I changed it to be permissive
- Firewall rules (had to add the firewall port too)
- Had to add the DNS there (not sure why)

This made the service available in my local network. After that I decided to expose it via cloudflare tunnel, by interacting with Gemini, I got the following:

{% highlight bash %}
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    environment:
      - TUNNEL_TOKEN=<cloudflaredtunneltoken>
    command: tunnel run
    network_mode: host
{% endhighlight %}

Accessed the cloudflare web UI, created a tunnel, added it in a domain like navi.home.com pointing to my home IP:4533 and it worked flawlessly. Later had to configure my Flo app to use my home server too, but well. This was quite simple

