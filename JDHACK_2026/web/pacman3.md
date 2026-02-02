# No share - Level 3
## Introduction
After multiple breaches of their file sharing system, our enemies have implemented wider security measures. This third iteration of their defense system now features refined filtering mechanisms that target specific IP address patterns. The administrators believe they have finally closed all loopholes by disabling DNS resolutions.
```python
# DNS resolution is now disabled
def deny(share):
     return 'secret.pacman' in share.lower() or 'localhost' in share or '.1' in share or ':' in share
```
## Challenge Resolution
As they now deactivated **DNS resolution**, we cannot use other hostname such as `lvh.me` anymore and have to use **IP addresses**. However, they do not filter the `1` but the `.1` which is in fact a relief for us because it is actually possible to represent IP addresses without points. 

Indeed, an IP address is nothing else than a **`32-bit integer`**, which can be represented as 4 decimal 8-bit integers (e.g. `127.0.0.1`) but also in **hexadecimal** (e.g. `0x7F000001`).

After a try with `Burp Repeater`, we confirm the hexadecimal representation works, and use `match and replace` as before to gain access to the `secret.pacman` share, and find the flag:
```flag
JDHACK{d0t_0n3_f1lt3r_byp4ss3d}
```