---
title: "Portswigger Web Academy: Host Header Attacks"
date: 2025-12-02 # YYYY-MM-DD
description: "Desc Text."
# weight: 1
# aliases: ["/first"]
draft: true
series: ["Portswigger Web Academy"]
categories: ["PSWA", "Web App Security"]
tags: ["appsec", "pswa", "websec", "hacking"]
showToc: true
TocOpen: false
hidemeta: false
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: true
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
    cover.responsiveImages: true
---

## 0x00: Intro & Recap

Labs: https://portswigger.net/web-security/all-labs#http-host-header-attacks

## 0x01: Basic Password Reset Poisoning

Prompt:
>  This lab is vulnerable to password reset poisoning. The user carlos will carelessly click on any links in emails that he receives. To solve the lab, log in to Carlos's account.  
> 
> You can log in to your own account using the following credentials: wiener:peter. Any emails sent to this account can be read via the email client on the exploit server. 

password reset, see how the request looks
can just mod the host header to point to our domain - gets concat'd to the domain in the email
carlos tries to access, 404
use the token in the URL to reset password
login -> success

## 0x02: Host Header Authentication Bypass

asdf

## 0x03: Web Cache Poisoning via Ambiguous Requests

asdf

## 0x04: Routing-based SSRF

asdf

## 0x05: SSRF via Flawed Request Parsing

asdf

## 0x06: Host Validation Bypass via Connection State Attack

asdf

## 0x07: Password Reset Poisoning via Dangling Markup

asdf
