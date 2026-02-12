---
title: "Portswigger Web Academy: Web Cache Poisoning"
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

Labs: https://portswigger.net/web-security/all-labs#web-cache-poisoning

## 0x01: Web cache poisoning with an unkeyed header

Solve date: 30 Dec, 2025

Prompt:
> This lab is vulnerable to web cache poisoning because it handles input from an unkeyed header in an unsafe way. An unsuspecting user regularly visits the site's home page. To solve this lab, poison the cache with a response that executes alert(document.cookie) in the visitor's browser.

Hint:
> This lab supports the X-Forwarded-Host header.



## 0x02: Web cache poisoning with an unkeyed cookie

Solve date: 31 Dec, 2025

Prompt:
> This lab is vulnerable to web cache poisoning because cookies aren't included in the cache key. An unsuspecting user regularly visits the site's home page. To solve this lab, poison the cache with a response that executes alert(1) in the visitor's browser.


After browsing around, we get 2 cookies: `session` and `fehost`
-> `fehost` is unkeyed

First stop: tried an `<svg onload>`
Next: can we close the bracket?
Fail.js
Final payload

## 0x03: Web cache poisoning with multiple headers

Prompt:
> This lab contains a web cache poisoning vulnerability that is only exploitable when you use multiple headers to craft a malicious request. A user visits the home page roughly once a minute. To solve this lab, poison the cache with a response that executes alert(document.cookie) in the visitor's browser.

Hint:
> This lab supports both the X-Forwarded-Host and X-Forwarded-Scheme headers.

Gotta send 2x poison to make sure - visitor only pops once a minute

Key takeaway: you can hijack JS files - don't forget about them during enum!

## 0x04: Targeted web cache poisoning using an unknown header

Prompt:
> This lab is vulnerable to web cache poisoning. A victim user will view any comments that you post. To solve this lab, you need to poison the cache with a response that executes alert(document.cookie) in the visitor's browser. However, you also need to make sure that the response is served to the specific subset of users to which the intended victim belongs.

Note the `Vary: User-Agent` header in response
When I started, this one is very unclear - how do I hook up the poison with the victim? What does the chain look like?
Does it go poison resource -> comment points to resource -> profit?

Insert img tag that points to exploit server to get the victim's UA: `Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36`

## 0x05: Web cache poisoning via an unkeyed query string

## 0x06: Web cache poisoning via an unkeyed query parameter

## 0x07: Parameter cloaking

## 0x08: Web cache poisoning via a fat GET request

## 0x09: URL normalization

## 0x0A: Web cache poisoning to exploit a DOM vulnerability via a cache with strict cacheability criteria

## 0x0B: Combining web cache poisoning vulnerabilities

## 0x0C: ache key injection

## 0x0D: Internal cache poisoning