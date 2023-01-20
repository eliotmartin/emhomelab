---
title: "Creating a Cloudflare Tunnel"
date: 2022-05-05T21:08:09+01:00
# weight: 1
# aliases: ["/first"]
tags: ["cloudflare","security"]
categories: ["tech"]
author: "Eliot"
showToc: true
TocOpen: false
draft: false
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
Exposing some of my services to the internet involves forwarding ports on my router (specifically 80 and 443) to the machines hosting them.  This is bad and makes me paranoid.  I wanted to be able to reach these services from outside without compromising my networks security by setting up a **Cloudflare Tunnel** and **NGINX Proxy Manager**

---
### Cloudflare's Tunnel
According to Cloudflare themselves, a **Cloudflare Tunnel**...

>  *...exposes applications running on your local web server on any network with an internet connection without manually adding DNS records or configuring a firewall or router.*

You can read more about *Cloudflare's Tunnels* on [their website](https://www.cloudflare.com/en-gb/products/tunnel/).  You can create a [free Cloudflare account](https://www.cloudflare.com/plans/free/) that allows you to create a Tunnel for no extra charge.

You will need to setup a domain on cloudflares DNS service for the Tunnel to work, but the domain itself doesn't need to be registered with Cloudflare.  I used one I bought from [Ionas for £1.00](https://www.ionos.co.uk/domains/domain-names), but it works just as well with free domains.  You just need to delgate Cloudflare as the authoritative DNS nameserver by replacing your domains original nameservers with ones provided by Cloudflare.

---
### NGINX Proxy Manager
The other component of this solution is a reverse proxy server, specifically in this case NGINX Proxy Manager.  **A reverse proxy server**...

> *...typically sits behind the firewall in a private network and directs client requests to the appropriate backend server. A reverse proxy provides an additional level of abstraction and control to ensure the smooth flow of network traffic between clients and servers.*

You can find more about *NGIX Proxy Manger* on [their website](https://nginxproxymanager.com/).  It is a free. open source solution. I setup NGIX in a docker container using Docker Compose, but that is probably worthy of a post of it own so I won't cover it here.

---
## Setting Up the cloudflare Tunnel
Cloudflare Tunnel requires the installation and configuration of a lightweight, open-source server-side daemon, *cloudflared*, to connect your infrastructure to Cloudflare.  Releases can be found on [GitHub](https://github.com/cloudflare/cloudflared/releases) and downloads are available as standalone binaries or packages like Debian and RPM.

### 1. Install the Cloudflared package

Download and setup Cloudflared using...

```lang-bash
$ wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb
$ sudo dkpg -i cloudflared-linux-arm64.deb
```
This creates a default directory when storing credentials files for your tunnel and the certificates it generates when you authenticate with Cloudflare.  The default directory is also where cloudflared will look for a configuration file (assuming no other file path is specified when running a tunnel).

This directory will be in the home directory `~/.cloudflared`.


### 2. Authenticate with Your Cloudflare Account
Cloudflared needs to be authenticated with a valid Cloudflare account using...

```lang-bash
$ cloudflared tunnel login
```
Open the URL that this generates and log into your Cloudflared account.  If successful the webpage will tell you everything is authenticated and will issue you a certificate by putting the file `cert.pem` in the `~/.cloudflared` directory.

### 3. Create a New Tunnel
With the Cloudflared successfully authenticate, you can create a tunnel...

```lang-bash
$ cloudflared tunnel create <TUNNEL-NAME>
```
- `<TUNNEL-NAME>` can be anything you want to call the new Tunnel

This sets up a new Tunnel (with the name `<TUNNEL-NAME>`) and creates a [Credentials file](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms#credentials-file) in the `~/.cloudflared` directory.  This can be confirmed with the command...

```lang-bash
$ cloudflared tunnel list
```

If all is well you should get an output along the lines of...

```lang-bash
You can obtain more detailed information for each tunnel with `cloudflared tunnel info <name/uuid>`
ID                                      NAME            CREATED                 CONNECTIONS
c522d8e5f-3b55-4abcd-8416-0335c87a1457  <TUNNEL-NAME>   2022-05-03T20:31:57Z    2xAMS, 2xLHR
```
***Note** `ID` in the reponse is the [Tunnel UUID](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms#tunnel-uuid).  You will need this later on.

### 4. Create a config.yaml File
Cloudflared is going to need some configuration to tell it how the Tunnel should function.  This is done using a `.yaml` file.  Cloudflared will automatically look for the configuration file in the default `~/.cloudflared` directory.  Create the file using...

```lang-bash
$ nano ~/.cloudflared/config.yaml
```
The `config.yaml` file should contain...
```yaml
tunnel: <TUNNEL-UUID>
credentials-file: /home/username/.cloudflared/<TUNNEL-UUID>.json
originRequest:
  originServerName: <DOMAIN>
ingress:
  - hostname: <DOMAIN>
  service: https://localhost:443
  originRequest:
    noTLSVerify: true
  - service: http_status:404
```

- `<TUNNEL-UUID>` is the [Tunnel UUID](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms#tunnel-uuid) you got earlier
- `<DOMAIN>` is the domain name you have setup in your Cloudflare DNS service

Adding additional subdomains to the `config.yaml` file is done by adding the following block...

```yaml
  - hostname: <SUBDOMAIN.DOMAIN>
  service: https://localhost:443
  originRequest:
    noTLSVerify: true
```
- `<SUBDOMAIN.DOMAIN>` is the subdomain name you have setup in your Cloudflare DNS service

Make sure the last `ingress:` entry in the `config.yaml` file is...

```yaml
  - service: http_status:404
```
### 5. Route DNS
For the Tunnel to work a CNAME recordin your Cloudflares DNS service needs to be created.  Run the command...

```lang-bash
$ cloudflared tunnel route dns <TUNNEL-UUID> <DOMAIN>
```
- `<TUNNEL-UUID>` is the [Tunnel UUID](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms#tunnel-uuid)
- `<DOMAIN>` is the domain name you have setup in your Cloudflare DNS service

### 6. Start the Tunnel
Almost there, fire up the Tunnel with the command...

```lang-bash
$ cloudflared tunnel run <TUNNEL-UUID>
```
- `<TUNNEL-UUID>` is the [Tunnel UUID](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-useful-terms#tunnel-uuid)

This command launches the Tunnel based on the configuration you specified in the `~/.cloudflared/config.yaml` file and produces and output similar to...

```lang-bash
INFO[2022-05-03T04:20:00] INF connection a513d3a2a-1b41-1dada-8a4a-1435a87b1800 registered connIndex=0 location=ATL
INFO[2022-05-03T04:20:01] INF connection b424c1c1a-2b33-2cbdf-7a3d-5435a87b1774 registered connIndex=0 location=IAD
INFO[2022-05-03T04:20:02] INF connection c335a2c1e-3a45-3bbde-6a1e-3453a87b1666 registered connIndex=0 location=ATL
INFO[2022-05-03T04:20:03] INF connection d246b7a4a-4a12-1afdf-5a2e-1635a87b1123 registered connIndex=0 location=IAD
```

All being well, you should be able to see the service at the end of the `<DOMAIN>`.  Press CTRL+C to stop the Tunnel.

### 8. Make Cloudflared a Service 
Now it's time to turn Cloudflared into a service so it starts up and runs in background whenever the host machine boots.  Run the command...

```lang-bash
$ sudo cloudflared service install
```
This *should* move all the contents of the `~/.cloudflared` directory into `/etc/cloudflared`.  Go to that directory, edit the `config.yaml` file tand point the `credentials-file` entry at the `/etc/cloudflared` directory...

```yaml
credentials-file: /etc/cloudflared/<TUNNEL-UUID>.json
```
Finally, enable, start, and check the service...

```lang-bash
$ sudo systemctl enable cloudflared
$ sudo systemctl start cloudflared
$ sudo systemctl status cloudflared
```

    
Thats that then!
