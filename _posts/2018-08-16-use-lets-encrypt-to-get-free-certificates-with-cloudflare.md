---
title: Use Let's Encrypt to get free certificates with Cloudflare
date: 2018-08-16 10:08:00
categories:
  - Tutorial
tags:
  - Let's Encrypt
  - HTTPS
  - Cloudflare
  - Certbot
  - homelab
---

[Chrome][1] and [Firefox][2] has planned to mark **ALL** the HTTP connection to be not secure.
"How insecure it can be?" you may asked.
Without HTTPS, HTTP is transferring the package in plain text, which is vulnerable to all kind of attack, especially Man-in-the-middle (MITM) attack.
For example, free Wi-Fi providers can modify any HTTP pages and inject their advertisements.
This can also happen when you send out data. The middle can steal and modify any data you send including user name and password.

<!--more-->

HTTPS used to cost money and time to get a certificate and the website owner has to keep renew it every several years.
This is now different because [Let’s Encrypt][3] is a free, automated, and open certificate authority (CA).
They are [sponsored by many big corporations][4] to increase the security of the web.

This tutorial is going to teach you how to get certificates from Let’s Encrypt using Certbot and Cloudflare.

# Requirements
  - Python 3 and pip installed (Basic understand of Python would be nice)
  - DNS record managed by Cloudflare

## Cloudflare

Cloudflare is a CDN provider and DNS provider. I use Cloudflare personal because it is free and provides many enterprise features.
I would not go too details in this tutorial. May be I will write a article about it in the future.

## Certbot

Certbot is a Python client that can work with ACME protocol to etches and deploys SSL/TLS certificates.

To install Certbot, please follow the [instructions from the official site][5].
For software you select `None of the above` if you want to install the certificates yourself.

For Ubuntu:

```bash
apt-get update
apt-get install software-properties-common
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install certbot 
```

To install the plugin for Cloudflare:

```bash
pip3 install certbot-dns-cloudflare
```

Run the following command to confirm you have installed correctly:

```bash
certbot plugins
```

# Get Certificate

First, you need to get your API key from Cloudflare.

Click "Get your API key"
![](https://user-images.githubusercontent.com/7152420/44191906-ddcd8980-a15f-11e8-990e-dbc9bb4f5555.PNG)

Click "View" of Global API Key
![](https://user-images.githubusercontent.com/7152420/44191908-de662000-a15f-11e8-9760-333e0117a740.PNG)

Copy your API key
![](https://user-images.githubusercontent.com/7152420/44191905-ddcd8980-a15f-11e8-9f57-7a390177aa32.PNG)

Create a `cloudflare.ini` and replace with your value.

```ini
dns_cloudflare_email = <Your Email>
dns_cloudflare_api_key = <Cloudflare API Key>
```

Run the following command:
```bash
certbot certonly \
  --renew-by-default \
  --server https://acme-v02.api.letsencrypt.org/directory \
  --dns-cloudflare \
  --dns-cloudflare-credentials ~/cloudflare.ini \
  --manual-public-ip-logging-ok \
  -m <Your Email> --no-eff-email \
  -d example.com -d *.example.com \
  --rsa-key-size 4096 \
  --agree-tos
```

* `certonly`: Certbot would not auto configure the web server for you but it only get the certificate.
* `renew-by-default`: Certbot will setup CRON job to auto renew.
* `server`: Only ACME v2 api can generate wildcard certificates.
* `dns-cloudflare`: Use Cloudflare plugin
* `dns-cloudflare-credentials`: Location of you ini file.
* `manual-public-ip-logging-ok`: Allow Let's Encrypt to log your ip.
* `m`: There will be email to notify your certificate is about to expire.
* `no-eff-email`: No email from EFF
* `d`: Valid domain(s) for these certificate. You should replace `example.com` with you domain. This first one will be the main domain. Also, `*` means wildcard.
* `rsa-key-size`: Size of the RSA key
* `agree-tos`: Agree term and services of Let's Encrypt

The certificate is available at `/etc/letsencrypt/live/<Your domain>` which is symlink to the latest certificate in `/etc/letsencrypt/archive/<Your domain>`.
Note that when you try to mount this in Docker.

Then, you can use the certificate whatever you want.

# Usage
Because the certificate is generated via DNS challenge, this means you can give a **valid** certificate for you internal domain. 

Also, if you have many different services, setting up Certbot for each services may be a problem.
Then, you can use a reverse proxy like nginx, Apache or traefik.
If you don't want to set it up yourself, you may consider using [nginx Docker][6] or [traefik Docker][7] which have Let's Encrypt already setup for you..

[1]: https://www.blog.google/products/chrome/milestone-chrome-security-marking-http-not-secure/
[2]: https://blog.mozilla.org/security/2017/01/20/communicating-the-dangers-of-non-secure-http/
[3]: https://letsencrypt.org/
[4]: https://letsencrypt.org/sponsors/
[5]: https://certbot.eff.org/
[6]: https://hub.docker.com/r/linuxserver/letsencrypt/
[7]: https://hub.docker.com/_/traefik/
