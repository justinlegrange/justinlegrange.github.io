---
title: "Portswigger Web Academy: Web Cache Poisoning"
date: 2025-12-02 # YYYY-MM-DD
lastMod: 2026-06-24
summary: "Solutions for Portswigger's Web Cache Poisoning academy module."
draft: true
series: ["Portswigger Web Academy"]
categories: ["PSWA", "hacking"]
tags: ["appsec", "pswa", "websec", "hacking"]
---

## 0x00: Intro & Recap

Labs: https://portswigger.net/web-security/all-labs#web-cache-poisoning

## 0x01: Web cache poisoning with an unkeyed header

> [!QUOTE]+ Problem Prompt
> This lab is vulnerable to web cache poisoning because it handles input from an unkeyed header in an unsafe way. An unsuspecting user regularly visits the site's home page. To solve this lab, poison the cache with a response that executes alert(document.cookie) in the visitor's browser.  
> Hint:  
> This lab supports the X-Forwarded-Host header.  
{icon="circle-question"}

Placeholder.

## 0x02: Web cache poisoning with an unkeyed cookie

> [!QUOTE]+ Problem Prompt
> This lab is vulnerable to web cache poisoning because cookies aren't included in the cache key. An unsuspecting user regularly visits the site's home page. To solve this lab, poison the cache with a response that executes alert(1) in the visitor's browser.
{icon="circle-question"}

After browsing around, we get 2 cookies: `session` and `fehost`
-> `fehost` is unkeyed

First stop: tried an `<svg onload>`
Next: can we close the bracket?
Fail.js
Final payload

## 0x03: Web cache poisoning with multiple headers

> [!QUOTE]+ Problem Prompt
> This lab contains a web cache poisoning vulnerability that is only exploitable when you use multiple headers to craft a malicious request. A user visits the home page roughly once a minute. To solve this lab, poison the cache with a response that executes alert(document.cookie) in the visitor's browser.
> Hint:
> This lab supports both the X-Forwarded-Host and X-Forwarded-Scheme headers.
{icon="circle-question"}

Gotta send 2x poison to make sure - visitor only pops once a minute

Key takeaway: you can hijack JS files - don't forget about them during enum!

## 0x04: Targeted web cache poisoning using an unknown header

> [!QUOTE]+ Problem Prompt
> This lab is vulnerable to web cache poisoning. A victim user will view any comments that you post. To solve this lab, you need to poison the cache with a response that executes alert(document.cookie) in the visitor's browser. However, you also need to make sure that the response is served to the specific subset of users to which the intended victim belongs.
{icon="circle-question"}

Note the `Vary: User-Agent` header in response
When I started, this one is very unclear - how do I hook up the poison with the victim? What does the chain look like?
Does it go poison resource -> comment points to resource -> profit?

Insert img tag that points to exploit server to get the victim's UA: `Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36`

## 0x05: Web cache poisoning via an unkeyed query string

Placeholder.

## 0x06: Web cache poisoning via an unkeyed query parameter

Placeholder.

## 0x07: Parameter cloaking

Placeholder.

## 0x08: Web cache poisoning via a fat GET request

Placeholder.

## 0x09: URL normalization

Placeholder.

## 0x0A: Web cache poisoning to exploit a DOM vulnerability via a cache with strict cacheability criteria

Placeholder.

## 0x0B: Combining web cache poisoning vulnerabilities

Placeholder.

## 0x0C: Cache key injection

Placeholder.

## 0x0D: Internal cache poisoning

Placeholder.
