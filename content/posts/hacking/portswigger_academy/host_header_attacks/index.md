---
title: "Portswigger Web Academy: Host Header Attacks"
date: 2025-12-02 # YYYY-MM-DD
lastMod: 2026-06-24
summary: "Solutions for Portswigger's Host Header Attacks academy module."
draft: true
# series: ["Portswigger Web Academy"]
# series_order: 1
categories: ["PSWA", "hacking"]
tags: ["appsec", "pswa", "websec", "hacking"]
---

## Introduction

Labs: https://portswigger.net/web-security/all-labs#http-host-header-attacks

## Challenge 1: Basic Password Reset Poisoning

> [!QUOTE]+ Problem Prompt
>  This lab is vulnerable to password reset poisoning. The user carlos will carelessly click on any links in emails that he receives. To solve the lab, log in to Carlos's account.  
> 
> You can log in to your own account using the following credentials: wiener:peter. Any emails sent to this account can be read via the email client on the exploit server. 
{icon="circle-question"}

password reset, see how the request looks
can just mod the host header to point to our domain - gets concat'd to the domain in the email
carlos tries to access, 404
use the token in the URL to reset password
login -> success

## Challenge 2: Host Header Authentication Bypass

Placeholder.

## Challenge 3: Web Cache Poisoning via Ambiguous Requests

Placeholder.

## Challenge 4: Routing-based SSRF

Placeholder.

## Challenge 5: SSRF via Flawed Request Parsing

Placeholder.

## Challenge 6: Host Validation Bypass via Connection State Attack

Placeholder.

## Challenge 7: Password Reset Poisoning via Dangling Markup

Placeholder.
