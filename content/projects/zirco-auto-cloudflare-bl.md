+++
title = "zirco-auto-cloudflare-bl"
description = "An experimental Matrix blocklist to educate on the harms of Cloudflare's orange cloud"
weight = 6

[extra]
github = "https://github.com/thetayloredman/zirco-auto-cloudflare-bl"
tags = ["matrix", "blocklist"]
+++

zirco-auto-cloudflare-bl is strongly inspired by [cs-open_reg-auto-acl](https://open-reg-list.codestorm.net/)'s
automated blocklist for open-registration Matrix servers.

This list scanned the Matrix federation automatically for servers that were using Cloudflare's orange cloud, and then automatically added them to a blocklist.

The goal was to educate on the many potential harms of using Cloudflare t
host Matrix servers, such as censorship risks.
