---
title: "MemLabs - Lab3"
classes: wide
header:
  teaser: /assets/images/Digital-Fronsic/memlabs/logo.png
ribbon: MidnightBlue
description: "A malicious script encrypted a very secret piece of information I had on my system. Can you recover the information for me please?
Note: This challenge is composed of only 1 flag and split into 2 parts
Hint: You'll need the first half of the flag to get the second..."
categories:
  - Digital Forensic
---

> ## **Challenge Description**
>
> A malicious script encrypted a very secret piece of information I had on my system. Can you recover the information for me please?
>
> **Note**: This challenge is composed of only 1 flag and split into 2 parts.
>
> **Hint**: You'll need the first half of the flag to get the second.
>
> You will need this additional tool to solve the challenge,
>
> ```
> $ sudo apt install steghide
> ```
>
> The flag format for this lab is: **inctf{s0me_l33t_Str1ng}**
>
> **Challenge file**: [MemLabs_Lab3](https://mega.nz/#!2ohlTAzL!1T5iGzhUWdn88zS1yrDJA06yUouZxC-VstzXFSRuzVg)


To deal with memory dump we need first to know operating system 
```
$ volatility -f MemoryDump_Lab3.raw imageinfo
```

[![1](/assets/images/Digital-Forensic/memlabs/lab3/1.jpg)](/assets/images/Digital-Forensic/memlabs/lab3/1.jpg)

Then we need to know the open processes during memory acquisition.

```
$volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86 pslist 
```
[![2](/assets/images/Digital-Forensic/memlabs/lab3/2.jpg)](/assets/images/Digital-Forensic/memlabs/lab3/2.jpg)

We see intersted process may be attacker use notpad.exe to run the malicious code .
so we will use plugin called cmdline 

```
$volatility -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 cmdline
************************************************************************
notepad.exe pid:   3736
Command line : "C:\Windows\system32\NOTEPAD.EXE" C:\Users\hello\Desktop\evilscript.py
************************************************************************
notepad.exe pid:   3432
Command line : "C:\Windows\system32\NOTEPAD.EXE" C:\Users\hello\Desktop\vip.txt
``` 

We found this .
next step we will use plugins called filescan to get the physical addresses.

```
$volatility -f MemoryDump_Lab3.raw --profile Win7SP1x86_23418 filescan | egrep "evilscript.py|vip.txt"
Volatility Foundation Volatility Framework 2.6                                                                                                                                                
0x000000003de1b5f0      8      0 R--rw- \Device\HarddiskVolume2\Users\hello\Desktop\evilscript.py.py
0x000000003e727490      2      0 RW-rw- \Device\HarddiskVolume2\Users\hello\AppData\Roaming\Microsoft\Windows\Recent\evilscript.py.lnk
0x000000003e727e50      8      0 -W-rw- \Device\HarddiskVolume2\Users\hello\Desktop\vip.txt
```
We will dump the two processes 


```
$ volatility -f MemoryDump_Lab3.raw --profile Win7SP1x86_23418 dumpfiles -Q 0x000000003de1b5f0 -D lab3_out/
$ volatility -f MemoryDump_Lab3.raw --profile Win7SP1x86_23418 dumpfiles -Q 0x000000003de1b5f0 -D lab3_out/
```

After dumping We notice that This evil script is XORing the file `vip.txt` with a single character then Base64 encoding it. 

[![3](/assets/images/Digital-Forensic/memlabs/lab3/3.jpg)](/assets/images/Digital-Forensic/memlabs/lab3/3.jpg)

we write simple python script to decode it .
```
>>> encodedCipher="am1gd2V4M20wXGs3b2U="
>>> encodedCipher= encodedCipher.decode("base64")
>>> flag = ''.join(chr(ord(i)^3) for i in encodedCipher)
>>> flag
'inctf{0n3_h4lf'
```
> #### First half: inctf{0n3_h4lf

Let's move on and find the other half, in description he gives us hint about steghide .
Steghide is a steganography program that is able to hide data in images and audio files and it supports JPEG and BMP images, so I decided to search memory for JPEG images.

```
$ volatility -f MemoryDump_Lab3.raw --profile Win7SP1x86_23418 filescan | grep ".jpeg"
Volatility Foundation Volatility Framework 2.6.1
0x0000000004f34148      2      0 RW---- \Device\HarddiskVolume2\Users\hello\Desktop\suspision1.jpeg
```
We found the only one image and it looks suspisious, let's dump it.

```
$volatility -f MemoryDump_Lab3.raw --profile Win7SP1x86_23418 dumpfiles -Q0x0000000004f34148 -D lab3_out/
Volatility Foundation Volatility Framework 2.6
DataSectionObject 0x04f34148   None   \Device\HarddiskVolume2\Users\hello\Desktop\suspision1.jpeg
```
[![4](/assets/images/Digital-Forensic/memlabs/lab3/4.jpeg)](/assets/images/Digital-Forensic/memlabs/lab3/4.jpg)


Here comes `steghide`, this image must have something hidden.

```
$ steghide extract -sf lab3_out/suspisious.jpeg 
Enter passphrase:
```

It's asking for a passphrase,  the hint clearly says that: `You'll need the first half of the flag to get the second`.

Let's try the first half of the flag as the passphrase.

```
$ steghide extract -sf lab3_out/suspisious.jpeg
Enter passphrase: `inctf{0n3_h4lf`
wrote extracted data to "secret text".
```

Voila!!! let's get this secret text.

```
$ cat secret\ text 
_1s_n0t_3n0ugh}
```

> #### Flag: inctf{0n3_h4lf_1s_n0t_3n0ugh}