---
title: "SACC - Smart AWS Cli Config"
draft: false
date: 2021-03-18T19:39:28+02:00
---

**S**mart **A**WS **C**li **C**onfig - `~/.aws/credentials` generator for lazy devs

project homepage: https://blog.ivnilv.com/tools/sacc/

project source code: https://github.com/ivnilv/sacc/

---

`sacc` is a tool built out of the necessity to get new AWS session tokens every 36 (maximum) hours. 

The tool will use your AWS access key and secret to request new session token from the AWS Security Token Service (AWS STS).

Usage:

![usage](/images/sacc.svg)

Prerequisites:

- You will need an existing AWS account
- MFA (Multi-factor authentication) device linked to your account
- AWS API Access key generated from your account

Requirements:
- bash 3.2+
- [jq](https://github.com/stedolan/jq)
- [aws-cli](https://github.com/aws/aws-cli)

Installation:

Clone the repository locally:
{{< highlight bash >}}
$ git clone https://github.com/ivnilv/sacc.git ~/sacc/
{{< / highlight >}}
Install a link to your `$PATH`:
{{< highlight bash >}}
$ sudo ln -s ~/sacc/sacc /usr/local/bin/
{{< / highlight >}}
