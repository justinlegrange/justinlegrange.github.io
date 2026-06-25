---
title: "Portswigger Web Academy: Host Header Attacks"
date: 2025-12-02 # YYYY-MM-DD
lastMod: 2026-06-24
summary: "Solutions for Portswigger's Host Header Attacks academy module."
draft: true
series: ["Portswigger Web Academy"]
categories: ["PSWA", "hacking"]
tags: ["appsec", "pswa", "websec", "hacking"]
---

## 0x00: Intro & Recap

Labs: https://portswigger.net/web-security/all-labs#http-host-header-attacks

## 0x01: Basic Password Reset Poisoning

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

## 0x02: Host Header Authentication Bypass

Placeholder.

## 0x03: Web Cache Poisoning via Ambiguous Requests

Placeholder.

## 0x04: Routing-based SSRF

Placeholder.

## 0x05: SSRF via Flawed Request Parsing

Placeholder.

## 0x06: Host Validation Bypass via Connection State Attack

Placeholder.

## 0x07: Password Reset Poisoning via Dangling Markup

Placeholder.
