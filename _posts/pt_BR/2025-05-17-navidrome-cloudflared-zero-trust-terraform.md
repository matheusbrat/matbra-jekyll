---
layout: post
title: Criando terraform para configurar cloudflared zero trust e compartilhar o navidrome com meu celular
lang: pt_BR
description: Criando terraform para configurar cloudflared zero trust e compartilhar o navidrome com meu celular
tags: navidrome spotify cloudflare terraform
comments: true
---


Olá,

No post anterior, eu configurei o Navidrome + Cloudflared para ouvir minhas músicas em casa. Decidi dar um passo adiante e adicionar a configuração do Cloudflare (túnel de confiança zero) para meu servidor doméstico usando Terraform. Decidi fazer isso usando o Gemini para ver como seria, quão complexo seria e quantos erros cometeria. Não estou profundamente familiarizado com o Terraform, então acho que isso tornou minha vida um pouco mais fácil, mas tive que fazer alguns ajustes manuais, pois alguns dos recursos sugeridos estavam obsoletos.

Eu defini o `variables.tf` como:

{% highlight terraform %}

variable "cloudflare_account_id" {
  type        = string
  description = "Seu ID de Conta Cloudflare"
}

variable "cloudflare_api_token" {
  type        = string
  description = "Seu Token de API do Cloudflare"
}

variable "domain_name" {
  type        = string
  description = "Seu nome de domínio"
  default     = "matbra.com"
}

variable "tunnel_name" {
  type        = string
  description = "O nome do Túnel Cloudflare"
  default     = "matbra-home-tunnel"
}

variable "ip_range" {
  type        = string
  description = "O intervalo de IP da sua rede doméstica"
  default     = "192.168.68.0/24"
}

variable "tunnel_secret" {
  type = string
  description = "O segredo para o Túnel Cloudflare"
  sensitive = true #  Mark the variable as sensitive
}


variable "emails_allowed" {
  type = list(string)
  description = "List e-mail allowed"
  default = ["<replace>"]
}
{% endhighlight %}

Temos coisas bem básicas: ID da conta Cloudflare, token de API, nome de domínio, nome do túnel, intervalo de IP e o segredo do túnel.

Além disso, `outputs.tf`:

{% highlight terraform %}

output "tunnel_token" {
  description = "Token do Túnel Cloudflare"
  value = cloudflare_zero_trust_tunnel_cloudflared.home_tunnel.tunnel_token
  sensitive   = true
}

{% endhighlight %}


Simplesmente definindo o local onde teremos a saída para ser usada no meu docker-compose cloudflared.

Também temos os recursos no `main.tf`:

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
    email = var.emails_allowed
  }
}

{% endhighlight %}

Acredito que também tive que mudar algumas outras coisas no painel deles, para permitir que meu celular se conectasse à minha rede local quando conectado à VPN. Não me lembro exatamente o que era, mas bem, tenho certeza que é possível.
