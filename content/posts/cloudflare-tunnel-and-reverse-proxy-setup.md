---
title: "Cloudflare Tunnel and NGINX Proxy Manager Setup"
date: 2022-05-05T21:08:09+01:00
# weight: 1
# aliases: ["/first"]
tags: ["first"]
categories: ["non-tech"]
author: "Eliot"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Avoiding port forwards and securing my shit"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowRssButtonInSectionTermList: true
cover:
    image: "/argo_tunnel.png" # image path/url
    alt: "Argo Tunnel" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
---
## The Problem
Exposing some of my homelab services to the internet involves forwarding ports on my router (specifically 80 and 443) to the machines hosting them.  This is bad and makes me paranoid.  I wanted to be able to reach these services from outside without compromising my networks security.

## The Solution
Set up a **Cloudflare Tunnel** and **NGINX Proxy Manager**

### Cloudflare's Tunnel
According to Cloudflare themselves, a **Cloudflare Tunnel**...

>  ...exposes applications running on your local web server on any network with an internet connection without manually adding DNS records or configuring a firewall or router.

You can read more about *Cloudflare's Tunnels* on [their website](https://www.cloudflare.com/en-gb/products/tunnel/), but importantly you can create a [free Cloudflare account](https://www.cloudflare.com/plans/free/) that allows you to create a Tunnel for no extra charge.

You will need to setup a domain on cloudflares DNS service for the Tunnel to work, but the domain itself doesn't need to be registered with Cloudflare.  I used one I bought from [Ionas for £1.00](https://www.ionos.co.uk/domains/domain-names), but it works just as well with free domains.  You just need to delgate Cloudflare as the authoritative DNS nameserver by replacing your domains original nameservers with ones provided by Cloudflare.

### NGINX Proxy Manager
The other component of this solution is a reverse proxy server, specifically in this case NGINX Proxy Manager.  **A reverse proxy server**...

> ...typically sits behind the firewall in a private network and directs client requests to the appropriate backend server. A reverse proxy provides an additional level of abstraction and control to ensure the smooth flow of network traffic between clients and servers.

You can find more about *NGIX Proxy Manger* on [their website](https://nginxproxymanager.com/), but importantly it is a free. open source solution. I setup NGIX in a docker container using Docker Compose, but that is probably worthy of a post of it own.

## Setting Up the cloudflare Tunnel
Cloudflare Tunnel requires the installation and configuration of a lightweight, open-source server-side daemon, *cloudflared*, to connect your infrastructure to Cloudflare.  Releases can be found on [GitHub](https://github.com/cloudflare/cloudflared/releases) and downloads are available as standalone binaries or packages like Debian and RPM.

### 1. Install the Cloudflared package
```
$ wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb
$ sudo dkpg -i cloudflared-linux-arm64.deb
```



