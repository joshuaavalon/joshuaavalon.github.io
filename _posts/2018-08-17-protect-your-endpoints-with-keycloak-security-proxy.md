---
title: Protect your endpoints with Keycloak Security Proxy
date: 2018-08-17 14:17:00
categories:
  - Tutorial
  - Homelab
tags:
  - Keycloak
  - Docker
  - reverse proxy
---

When we setup applications in our homelab, there are always some applications do not have authentication or authentication that can integrated with you existing system.

<!--more-->

The solution I was using was [Organizr][1].
While it protects your endpoint, it is not the best solution.
If you read the [source code][2], it is just caching *IP address* which can lead to a lot of problems.
Also, it does not redirect you to login page or allow you to setup for remote authentication.
It only has limited (admin and user) role to control access and do not play well with other identity providers, like LDAP.

# Requirements
  - Keycloak (Admin privilege)
  - Docker
  - Docker Compose

## Keycloak

![Keycloak][4]

Keycloak is an open source identity and access management application. [It has integration with many useful services][3], including SAML, open ID, LDAP, single sign on, etc.

By using Keycloak, you now only have one place to manage user credentials and users only have to remember one account.  

## Docker

![Docker][5]

In this tutorial, we will use Docker to deploy because it is much easier to install and manage applications.

It is still possible to set it up without Docker, but that will not be covered in this tutorial.

## Docker Compose

![Docker Compose][6]

Making use of Docker Compose allow us no long need to remember the long docker command.
We can even put the file in a version control system.

# Let's Start

## Configure Keycloak
First, we login to Keycloak with a admin user.
Then, we create a client with `whoami` as Client ID (Any thing you like) and select `openid-connect` as Client Protocol.
We change the Access Type to `confidential`  and add your public url to Valid Redirect URIs, e.g. `http://example.com/*`. Remember you have to save it.
We go to Installation and select `Keycloak OIDC JSON` as Format Option.
We will need it when we configure Keycloak Security Proxy.


## Configure Keycloak Security Proxy
We will create a configuration file at `/home/user/proxy/proxy.json`.

```json
{
  "target-url": "http://whoami",
  "bind-address": "0.0.0.0",
  "http-port": "8080",
  "applications": [
    {
      "base-path": "/",
      "adapter-config": {
        "realm": "master",
        "auth-server-url": "http://example.com/auth",
        "ssl-required": "external",
        "resource": "test",
        "credentials": {
          "secret": "6c394787-d92d-46a6-98c9-960a6c2e98c7"
        },
        "confidential-port": 0
      },
      "constraints": [
        {
          "pattern": "/*",
          "roles-allowed": ["user"]
        }
      ]
    }
  ]
}
```
All the configuration can be found [here][7].

**target-url:** This the address to proxy. We use `whoami` because we are going to to deploy a container with a hostname of `whoami`.
**bind-address:** It needs to be `0.0.0.0` in order to listen request from outside.
**http-port:** The port that it listen to. This does not matter as we have to map it on Docker again.
**base-path:** Base path of the application. Since we only have 1 application, we use `/`.
**adapter-config:** This is generated from Keycloak. Paste the configuration from previous section. 
**constraints:** Constraints for accessing the application. In this example, all paths can only be access if they have have `user` role.

## Configure Docker Compose

Then, we create the following `docker-compose.yml`.

```yml
version: "3"

services:
  whoami:
    image: emilevauge/whoami
  proxy:
    image: jboss/keycloak-proxy
    ports:
      - 80:8080
    volumes:
      - /home/user/proxy:/opt/jboss/conf
```

## Run it
We start it up with 

```bash
docker-compose up -d
```

When you try to access it without login, it will redirect you to Keycloak login page.
If you logined but you do not match the constraints, you will get a 403 response.
If you have logined and you match the constraints, you can just use it normally.

[1]: https://github.com/causefx/Organizr
[2]: https://github.com/causefx/Organizr/blob/4312e360d9a0291d20b7a776897ee63e997ab20f/auth.php
[3]: https://www.keycloak.org/about.html
[4]: https://user-images.githubusercontent.com/7152420/44251372-b1cb0a80-a22a-11e8-9e58-68d3c1da2822.png
[5]: https://user-images.githubusercontent.com/7152420/44251743-189cf380-a22c-11e8-9ac2-7d52c08f58e8.png
[6]: https://user-images.githubusercontent.com/7152420/44251782-4124ed80-a22c-11e8-8364-7e0ebd30eb80.png
[7]: https://www.keycloak.org/docs/3.3/server_installation/topics/proxy.html