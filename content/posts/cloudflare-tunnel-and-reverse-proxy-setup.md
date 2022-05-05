---
title: "Cloudflare Tunnel and Reverse Proxy Setup"
date: 2022-05-05T21:08:09+01:00
# weight: 1
# aliases: ["/first"]
tags: ["first"]
categories: ["non-tech"]
author: "Eliot"
showToc: false
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "REPLACEME"
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
I got worried about forwarding ports from my router to the homelab servers inside my network so did some digging on how I could better secure my infrastructure.  

This is what I discovered and how I got it working.

## Cloudflare's Tunnel
According to Cloudflare themselves, a *Cloudflare Tunnel*...

>  ...exposes applications running on your local web server on any network with an internet connection without manually adding DNS records or configuring a firewall or router.

You can (read more about them on their [website](https://www.cloudflare.com/en-gb/products/tunnel/), but importantly you can create a [free Cloudflare account](https://www.cloudflare.com/plans/free/)and create an Cloudflare Tunnel for no extra charge.
### Pre-requisites
You need to setup a domain on cloudflares DNS service under your account.

The domain itself doesn't even need to be registered with Cloudflare.  I used one I bought from [Ionas for a quid](https://www.ionos.co.uk/domains/domain-names), but it works just as well with free domains.  All that you need to do is delgate Cloudflare as the authoritative DNS nameserver by...

1. logging into my cloudflare dashboard

## Reverse Proxy


