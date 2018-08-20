---
title: Why you should use Cloudflare and when you should not
date: 2018-08-20 10:30:00
categories:
  - Article
tags:
  - Cloudflare
  - homelab
---

[When I previously show you show to get a valid certificate with Certbot, I have talked about Cloudflare][1].
Today, I am going to tell you why you should use Cloudflare.
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



[1]: {{ site.baseurl }}{% post_url 2010-07-21-name-of-post %}
[2]: https://cloudflare.com