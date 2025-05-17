---
layout: post
title: Creating terraform to setup cloudflared zero trust and share navidrome to my phone
lang: en
description: Creating terraform to setup cloudflared zero trust and share navidrome to my phone
tags: navidrome spotify cloudflare terraform
comments: true
---


Hello,

In the previous post I did the setup of Navidrome + Cloudflared to listen my home musics, I decided to take one step further and add the setup of cloudflare (zero trust tunnel) for my home server using terraform. I decided to do it using Gemini and see how it would go, how complex it would be, how many mistakes. I'm not deeply familiar with terraform thus I think it made my life slightly easier but I had to do some manual adjustments as some of the suggested resources were deprecated. 

I defined the variables.tf as:

{% highlight terraform %}

variable "cloudflare_account_id" {
  type        = string
  description = "Your Cloudflare Account ID"
}

variable "cloudflare_api_token" {
  type        = string
  description = "Your Cloudflare API Token"
}

variable "domain_name" {
  type        = string
  description = "Your domain name"
  default     = "matbra.com"
}

variable "tunnel_name" {
  type        = string
  description = "The name of the Cloudflare Tunnel"
  default     = "matbra-home-tunnel"
}

variable "ip_range" {
  type        = string
  description = "The IP range of your home network"
  default     = "192.168.68.0/24"
}

variable "tunnel_secret" {
  type = string
  description = "The secret for the Cloudflare Tunnel"
  sensitive = true #  Mark the variable as sensitive
}

{% endhighlight %}

We have very basic stuff: cloudflare account id, api token, domain name, tunnel name, ip range, and the tunnel secret. 

Besides it, outputs.tf

{% highlight terraform %}

output "tunnel_token" {
  description = "Cloudflare Tunnel Token"
  value = cloudflare_zero_trust_tunnel_cloudflared.home_tunnel.tunnel_token
  sensitive   = true
}

{% endhighlight %}


Simply defining the place where we will have the output to be used in my docker-compose cloudflared.

We also have the resources in main.tf

{% highlight terraform %}

terraform {
  required_providers {
    cloudflare = {
      source  = "cloudflare/cloudflare"
      version = "~> 4.0"
    }
  }
}

provider "cloudflare" {
  api_token = var.cloudflare_api_token
}

# Create a Cloudflare Tunnel. This sets up the connection point
# between your home network and Cloudflare's network.
resource "cloudflare_zero_trust_tunnel_cloudflared" "home_tunnel" {
  account_id = var.cloudflare_account_id
  name       = var.tunnel_name
  secret     = var.tunnel_secret
}

# Create a Cloudflare Tunnel route. This tells Cloudflare to route
# traffic for your home network's IP range through the tunnel.
resource "cloudflare_zero_trust_tunnel_route" "home_network_route" {
  account_id = var.cloudflare_account_id
  tunnel_id  = cloudflare_zero_trust_tunnel_cloudflared.home_tunnel.id
  network    = var.ip_range # Add the network argument here, use the ip_range variable
}

# Create a Cloudflare Access application for the general home network access.
resource "cloudflare_zero_trust_access_application" "home_network_app" {
  account_id = var.cloudflare_account_id
  name       = "Matbra Home Network Access Application"
  session_duration = "24h"
}

# Create a Cloudflare Zero Trust policy to control access to the resources
# behind the tunnel.
resource "cloudflare_zero_trust_access_policy" "home_network_policy" {
  application_id = cloudflare_zero_trust_access_application.home_network_app.id
  account_id = var.cloudflare_account_id
  name       = "Matbra Home Network Access Policy"
  precedence = 1
  decision   = "allow"

  include {
    email = ["matheusbrat@gmail.com"]
  }
}

{% endhighlight %}

I believe I also had to change some other stuff in their dashboard, to allow my phone to connect to my local network when connected to the vpn. I don't remember exactly what it was, but well, I'm sure it is possible. 

