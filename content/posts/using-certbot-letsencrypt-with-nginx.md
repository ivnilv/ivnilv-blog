---
title: "Using Certbot Letsencrypt With Nginx"
date: 2021-03-11T16:22:58+02:00
draft: false
tags:
- Linux
- certbot
- nginx
- letsencrypt
---


#### Install CertBot

Installing `certbot` in Ubuntu is pretty straight forward:

{{< highlight bash >}}
# apt-get install certbot
{{< / highlight >}}

#### Generate certificate

Based on the type of webserver you are using, you need to pass a different parameter to the `certbot` in order for it to properly configure your virtual host.

In the case of nginx , it's as simple as:


{{< highlight bash >}}
# certbot --nginx
{{< / highlight >}}

Next, you'll need to answer a couple of questions which are necessary for the certificate issuance. Those questions include your email address, for which vhost you would like to have SSL certificate issued (given you have additional vhosts), and whether should certbot set a redirect for your HTTP traffic to HTTPS.

