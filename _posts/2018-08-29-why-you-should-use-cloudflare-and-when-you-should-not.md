---
title: Why you should use Cloudflare and when you should not
date: 2018-08-29 10:52:00
categories:
  - Article
tags:
  - Cloudflare
  - homelab
---

[When I previously show you show to get a valid certificate with Certbot, I have talked about Cloudflare][1].
Today, I am going to tell you why you should use [Cloudflare][Cloudflare].
There are many useful features that you can use even are on the free plan.

<!--more-->

# Create Account

First, you need to register on Cloudflare.
After you created your account, you need to let Cloudflare manage your domain.
You can do that by adding you domain to Cloudflare and following the instructions.
Make sure all the DNS record is imported correctly.
Basically, it is just go to your domain registration service and point the name server to Cloudflare name server.
The changes may take some time to propagate.

# Using Cloudflare

To use Cloudflare, you have to ensure the A / AAAA / CNAME record in the *DNS* tab is enabled (orange cloud).

## Overview

After you enable Cloudflare, others can no longer see your server IP because all the traffics go through Cloudflare.
This hides your server from attack.
Cloudflare also provides a **valid** certificate for you and cache your files.

Let's go through each section in Cloudflare. Note that I will only be talk about the features available on **free* plan.

## Analytics

This section shows you what traffic and how many traffic are going to your server, including countries, bots and threats.

## DNS

This section manages your DNS records.
Cloudflare supports nearly all kind of DNS record from A, AAAA, CNAME to TLSA, SSHFP.
You can have wildcard record but they cannot go through Cloudflare on free plan.
But the should not be a problem because you can have update to 500 records per domain by default and you can contact support if you need more.

Also, Cloudflare supports [DNSSEC][DNSSEC] which protect your DNS records.

## Crypto

This section manages cryptography settings for your websites.

### SSL

This controls how Cloudflare connect to your server and how Cloudflare connect to visitors.

* Off: Visitor - HTTP -> Cloudflare - HTTP -> server
* Flexible: Visitor - HTTPS -> Cloudflare - HTTP -> server
* Full: Visitor - HTTPS -> Cloudflare - HTTPS (Allow invalid) -> server
* Full (Strict): Visitor - HTTPS -> Cloudflare - HTTPS -> server


Note that this part is very important. If you misconfigure it, it will result in 526 error.

First, you should **NEVER** use `Off`, under no circumstances. It have no benefits over this other options. 

`Flexible` should only be used if you server cannot setup HTTPS which is seldom the case because you can always use self-signed certificate.
If you web application cannot use a certificate, you can setup a reverse proxy to do that. 
It is not recommended to use this because traffic between you and Cloudflare is unencrypted and allows MITM attacks.

`Full` should only be used if you are using invalid certificate (i.e. self-signed).
It is easy to get a valid certificate with Let's Encrypt. You can use DNS challenge which means even you server cannot be direct access by Let's Encrypt, you can still get a valid certificate.
It is also not recommended to use this because traffic between you and Cloudflare still allows MITM attacks.

`Full (Strict)` is recommended because all traffic is encrypted under valid certificate.


### Others

You can enable `Always use HTTPS` which is basically response 301 redirect to HTTPS on all HTTP request. I recommend to enable it.

Cloudflare also provides HTTP Strict Transport Security (HSTS), Authenticated Origin Pulls, Opportunistic Encryption, and Automatic HTTPS Rewrites.

## Firewall

This section manages what traffic to block or allow to your website. You can set it by IP range, country, ASN, user agent.
You can also enable DDOS protection when you are under attack. There are logs for firewall events.

## Access

This section allows you to create additional access layer to control traffic to you website on different path.
For example, dev.example.com/test can only be access if the user has a Gsuite Business account of @example.com.
You can connect it to Oauth or SAML provider. Note that it is only free up to 5 users.

## Speed

This section providing multiple functions to speed up you website, including auto minify CSS, javascript, HTML.
However, you should test Rocket Loader before enable it. I tried multiple web applications break when it is enabled.

## Caching

This section manages how Cloudflare cache the files, including cache time and cache purging.

## Page Rules

This section allows to have the above setting based the address instead of apply to you whole domain.
But can only have 3 page rules for free.

# Conclusion

It is definitely worth trying out Cloudflare. Not only it is free but it provides many functions and has a much better UI than most of the domain registration service. 

If you are a programmer, you can also control the setting through REST API.

[1]: {{ site.baseurl }}{% post_url 2010-07-21-name-of-post %}
[Cloudflare]: https://cloudflare.com
[DNSSEC]: https://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions