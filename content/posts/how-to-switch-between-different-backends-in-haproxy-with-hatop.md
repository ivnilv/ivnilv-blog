---
title: "How to Switch Between Different Backends in Haproxy Using Hatop"
date: 2021-03-14T13:37:34+02:00
draft: false
tags:
- Linux
- haproxy
- loadbalancing
- hatop
---

Let's assume the following setup where we have a HAproxy frontend accepting incoming requests for an app in port 80, and then forwarding those requests to the application's backend servers (nginx web instances).

This would be useful, for example when you would like to upgrade the version of nginx servers hosting your web application's code to the latest version of nginx with **zero downtime** !

Here is the minimal `haproxy.cfg` configuration file we are going to use for this guide:

```
backend bk_app # define a backend for the app containing a list of backend servers
  server nginx1 127.0.0.1:81 check backup    # this server is backup and does not serve clients
  server nginx2 127.0.0.1:82 check 

frontend app                          # define what port to listed to for HAProxy
  bind :80
  default_backend bk_app                  # set the default server for all request
``` 

![haproxy-nginx-scheme](/images/haproxy-nginx-scheme.png)

Currently, we have 2 nginx instances, one running the latest version of nginx 1.19.8 running on port 81; and another nginx instance on port 82 running nginx version 1.18.0.

As you can see, hitting port 80 will always get served by the older 1.18.0 instance:

![haproxy-nginx-check-1](/images/haproxy-nginx-1.gif)

Now in order to switch the clients to the backend server running the latest version, you will need to find the haproxy admin socket (it's usually configured in `/etc/haproxy/haproxy.cfg`) and start the `hatop` utility. An example command would look like:

```
root@ivnilv-vm-mobile:~# hatop -s /run/haproxy/admin.sock
```
This should lead you to the HAtop interface which should look something like:

![hatop-ui](/images/hatop1.png)

Using the `hatop` interactive cli you can navigate between the available backend servers in the list above using the up (↑) and down (↓) arrow keys. 

Switching the clients off of our nginx2 instance (1.18.0) to the nginx1 (1.19.8) instance would require selecting the "nginx2" backend server in the list > HIT "Enter" key > HIT "F10" key (DISABLE). 

What this would do is put the selected backend server into maintenance mode, thus start serving client requests from the backup backend server, in this case being the nginx1 (1.19.8) instance.

Let's do a live test on this and see the result.

In the below example, we are running a GET request to our HAproxy loadbalancer every 2 seconds. Meanwhile, we are going to put the nginx2 instance into maintenance mode, effectively making the nginx1 instance (1.19.8) the primary one/i.e. the one serving client requests:

![haproxy-nginx-check-1](/images/haproxy-nginx-2.gif)

And thats it ! You have successfully switched the backend server for your application to the latest version of your web server including the latest security and feature patches. **With zero downtime**.

Hope you enjoyed this article and you find it useful.

Leave any comments and replies in the section below and don't forget to subscribe for the push notifications to get the latest of what this blog has to offer 🤓

Thanks!