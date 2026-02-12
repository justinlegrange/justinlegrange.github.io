---
title: "Portswigger Web Academy: Information Disclosure"
date: 2026-02-05 # YYYY-MM-DD
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

Labs: https://portswigger.net/web-security/all-labs#information-disclosure

## 0x01: Information disclosure in error messages

Prompt:
> This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework. To solve the lab, obtain and submit the version number of this framework.

asdf

## 0x02 :: Information disclosure on debug page

> This lab contains a debug page that discloses sensitive information about the application. To solve the lab, obtain and submit the SECRET_KEY environment variable. 

asdf

## 0x03 :: Source code disclosure via backup files

> This lab leaks its source code via backup files in a hidden directory. To solve the lab, identify and submit the database password, which is hard-coded in the leaked source code.

asdf

## 0x04 :: Authentication bypass via information disclosure

Prompt:
> This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.  
>  
> To solve the lab, obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete the user carlos.  
> 
> You can log in to your own account using the following credentials: wiener:peter  

asdf

## 0x05 :: Information disclosure in version control history

Prompt:
> This lab discloses sensitive information via its version control history. To solve the lab, obtain the password for the administrator user then log in and delete the user carlos.

asdf