---
title : " MemLabs - Lab1"
classes : wide 
header :
  teaser: /assets/images/Digital-Forensic/memlabs/logo.png
ribbon: MidnightBlue
description: "MemLabs is an educational, introductory set of CTF-styled challenges which is aimed to encourage students, security researchers and CTF  players to get started with the field of Memory Forensics
Each challenge has a description along with a memory dump file. We are supposed to get all the flags using memory forensics tools (mainly volatility)..."
categories:
  - Digital Forensic
---

MemLabs is an educational, introductory set of CTF-styled challenges which is aimed to encourage students, security researchers and CTF  players to get started with the field of **Memory Forensics**.

Each challenge has a description along with a memory dump file. We are supposed to get all the flags using memory forensics tools (mainly volatility).

You can find MemLabs here :[MemLabs](https://github.com/stuxnet999/MemLabs)

> ## **Challenge Description**
> 
> My sister's computer crashed. We were very fortunate to recover this memory dump. Your job is get all her important files from the system. From what we remember, we suddenly saw a black window pop up with some thing being executed. When the crash happened, she was trying to draw something. Thats all we remember from the time of crash.
>
> **Note**: This challenge is composed of 3 flags.
>
> **Challenge file**: [MemLabs_Lab1](https://mega.nz/#!6l4BhKIb!l8ATZoliB_ULlvlkESwkPiXAETJEF7p91Gf9CWuQI70)

To deal with memory dump file first we need to know the operating system , for that we can use plugin called `imageinfo`

```
$ volatility -f MemoryDump_Lab1.raw imageinfo
```
[![1](/assets/images/Digital-Forensic/memlabs/lab1/1.jpg)](/assets/images/Digital-Forensic/memlabs/lab1/1.jpg)

Volatility has alot of suggested profile , often the first is suitable to check .

Next we need to know the running processes , so we use plugin called `pslist` ->list all open process  during memory acquisition.

```
$ volatility -f MemoryDump_Lab1.raw --profile Win7SP1x64 pslist
```
[![2](/assets/images/Digital-Forensic/memlabs/lab1/2.jpg)](/assets/images/Digital-Forensic/memlabs/lab1/2.jpg)

Reading challenge descriptionagain we notice `we suddenly saw a black window pop up with some thing being executed. When the crash happened, she was trying to draw something. Thats all we remember from the time of crash.` 

There are 3 interesting processes here, let's start with `cmd.exe`. this process indicates commands executes on system.

We can use `consoles` plugin to see the output 

```
$ volatility -f MemoryDump_Lab1.raw --profile Win7SP1x64 consoles
```

[![3](/assets/images/Digital-Forensic/memlabs/lab1/3.jpg)](/assets/images/Digital-Forensic/memlabs/lab1/3.jpg)

Look we found the output of this command `St4G3$1` is `base64` if we decode it we have the 1st flag .

```
$ echo ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0= | base64 -d
flag{th1s_1s_th3_1st_st4g3!!}
```

> #### Flag 1: flag{th1s_1s_th3_1st_st4g3!!}

Next we will focus on the second interesting process , which is `mspaint` . The PID of this process is 2424.

we can use plugin called `memdump` to extract the image back .

`memdump` take two arguments [prosess id , ouput directory]

```
$volatility -f MemoryDump_Lab1.raw --profile Win7SP1x64 memdump -p 2424 -D lab1_out/
```

The output is written to `2424.dmp`, we need to rename it to `2424.data` to be able to open it in Gimp.

After playing a bit with the width and offset. I got an image which is somewhat flipped. I rotated it 180 degrees then flipped it horizontally and Voila!, I got the flag.


[![4](/assets/images/Digital-Forensicctf-writeups/memlabs/lab1/4.jpg)](/assets/images/Digital-Forensic/memlabs/lab1/4.jpg) | [![5](/assets/images/Digital-Forensic/memlabs/lab1/5.jpg)](/assets/images/Digital-Forensic/memlabs/lab1/5.jpg)

> #### Flag 2: flag{G00d_Boy_good_girL}

Next we will focus on the second interesting process , which is `winrar.exe` . The PID of this process is 1512.

we can use plugin called `cmdline`->to see the associated command line.  or direct `fiescan`->to see psychical offset of that file in memory.


```
$ volatility -f MemoryDump_Lab1.raw --profile Win7SP1x64 filescan | grep Important.rar
Volatility Foundation Volatility Framework 2.6.1
0x000000003fa3ebc0      1      0 R--r-- \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
0x000000003fac3bc0      1      0 R--r-- \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
0x000000003fb48bc0      1      0 R--r-- \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
```

We can pick any of these offsets, To dump the file we can use `dumpfiles` plugin.

```
$ volatility -f MemoryDump_Lab1.raw --profile Win7SP1x64 dumpfiles -Q 0x000000003fa3ebc0 -D lab1_out/
Volatility Foundation Volatility Framework 2.6.1
DataSectionObject 0x3fa3ebc0   None   \Device\HarddiskVolume2\Users\Alissa Simpson\Documents\Important.rar
```

The file is dumped under the name `file.None.0xfffffa8001034450.dat`, let's rename and unrar it.


```
$ mv file.None.0xfffffa8001034450.dat Important.rar
$ unrar e Important.rar 

UNRAR 5.61 beta 1 freeware      Copyright (c) 1993-2018 Alexander Roshal

Extracting from Important.rar

Password is NTLM hash(in uppercase) of Alissa's account passwd.

Enter password (will not be echoed) for flag3.png: 
```

The file is password protected, but we can see a comment that says the password is the NTLM hash of Alissa's account passwd.

To get the password hash, we can use `hashdump` plugin.

```
$ volatility -f MemoryDump_Lab1.raw --profile Win7SP1x64 hashdump
Volatility Foundation Volatility Framework 2.6.1
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SmartNet:1001:aad3b435b51404eeaad3b435b51404ee:4943abb39473a6f32c11301f4987e7e0:::
HomeGroupUser$:1002:aad3b435b51404eeaad3b435b51404ee:f0fc3d257814e08fea06e63c5762ebd5:::
Alissa Simpson:1003:aad3b435b51404eeaad3b435b51404ee:f4ff64c8baac57d22f22edc681055ba6:::
```

The desired NTLM hash is `f4ff64c8baac57d22f22edc681055ba6` (remember it must be in uppercase).

```
$python
>>"f4ff64c8baac57d22f22edc681055ba6".upper()
'F4FF64C8BAAC57D22F22EDC681055BA6'
```

After decompressing the file, we get an image with the flag.
```
unrar e Important.rar 

UNRAR 6.00 freeware      Copyright (c) 1993-2020 Alexander Roshal


Extracting from Important.rar

Password is NTLM hash(in uppercase) of Alissa's account passwd.

Enter password (will not be echoed) for flag3.png: F4FF64C8BAAC57D22F22EDC681055BA6

Extracting  flag3.png                                                 OK 
All OK
```

[![6](/assets/images/Digital-Forensic/memlabs/lab1/6.png)](/assets/images/Digital-Forensic/memlabs/lab1/6.png)

> #### Flag 3: flag{w3ll_3rd_stage_was_easy}




