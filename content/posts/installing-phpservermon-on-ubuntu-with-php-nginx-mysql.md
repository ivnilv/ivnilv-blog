+++
title = "Install PHPServerMon on Ubuntu with PHP-FPM, nginx, MySQL"
date = "2021-04-23T20:24:43+03:00"
author = ""
authorTwitter = "" #do not include @
cover = "/images/lemp.png"
tags = ["monitoring", "linux", "install", "phpservermon"]
keywords = ["", ""]
description = ""
showFullContent = false
+++

# Install requirements

- Install git:

{{< highlight bash >}}
sudo apt install git -y
{{< /highlight >}}


- Install PHP 7.4

{{< highlight bash >}}
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt-get install -y php7.4
{{< /highlight >}}

- Install NginX 

{{< highlight bash >}}
sudo apt-get install nginx -y
{{< /highlight >}}
