---
title: "MemLabs - Lab2"
classes: wide
header:
  teaser: /assets/images/Digital-Forensic/memlabs/logo.png
ribbon: MidnightBlue
description: "One of the clients of our company, lost the access to his system due  to an unknown error. He is supposedly a very popular 'environmental'  activist. As a part of the investigation, he told us that his go to  applications are browsers, his password managers etc. We hope that you  can dig into this memory dump and find his important stuff and give it  back to us..."
categories:
  - Digital Forensic
---

> ## **Challenge Description**
>
> One of the clients of our company, lost the access to his system due  to an unknown error. He is supposedly a very popular "environmental"  activist. As a part of the investigation, he told us that his go to  applications are browsers, his password managers etc. We hope that you  can dig into this memory dump and find his important stuff and give it  back to us.
>
> **Note**: This challenge is composed of 3 flags.
>
> **Challenge file**: [MemLabs_Lab2](https://mega.nz/#!ChoDHaja!1XvuQd49c7-7kgJvPXIEAst-NXi8L3ggwienE1uoZTk)


To deal with memory dump file first we need to know suggested profile to know the operating system .


```
$ volatility -f MemoryDump_Lab2.raw imageinfo
```

[![1](/assets/images/Digital-Forensic/memlabs/lab2/1.jpg)](/assets/images/Digital-Forensic/memlabs/lab2/1.jpg)

Next, let's check the processes list.

```
$ volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 pslist
```

[![2](/assets/images/Digital-Forensic/memlabs/lab2/2.jpg)](/assets/images/Digital-Forensic/memlabs/lab2/2.jpg)

There are a lot of interesting processes such as`(chrome.exe, notepad.exe, keepass.exe , cmd.exe )` .

Note he said in the challenge description `environment` let's make some digging .

We will use plugin called `envars` -> show us all environment variables for each process .

```
$ volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 envars | less

``` 

[![3](/assests/images/Digital_Forensic/memlabs/lab2/3.jpg)](/assests/images/Digital_Forensic/memlabs/lab2/3.jpg)

We found base64 string we decode it we have the first flag

```
$ echo ZmxhZ3t3M2xjMG0zX1QwXyRUNGczXyFfT2ZfTDRCXzJ9 | base64 -d
flag{w3lc0m3_T0_$T4g3_!_Of_L4B_2}
```

Great, first stage is done.

> #### Flag 1: flag{w3lc0m3_T0_$T4g3_!_Of_L4B_2}

Next, let's check this `KeePass` process, looks like a password manager.

> **if you need to read about keepass **: [keepass](https://keepass.info/)

The extensoin of keepass file is : `kdbx`

```
$ volatility -f MemoryDump_Lab2.raw --profile=Win7SP1x64 filescan |grep kdbx
Volatility Foundation Volatility Framework 2.6
0x000000003fb112a0     16      0 R--r-- \Device\HarddiskVolume2\Users\SmartNet\Secrets\Hidden.kdbx

```
Let's dump it , we will use plugin called dumpfiles take two arguments [physical address , output folder]

```
$volatility -f MemoryDump_Lab2.raw --profile Win7SP1x64 dumpfiles -Q 0x000000003fb112a0 -D lab2_out/
Volatility Foundation Volatility Framework 2.6
DataSectionObject 0x3fb112a0   None   \Device\HarddiskVolume2\Users\SmartNet\Secrets\Hidden.kdbx
```
We try to find the master password so we use file scan to search for the `password`.

```
$volatility -f MemoryDump_Lab2.raw --profile Win7SP1x64 filescan | grep -i "password"
Volatility Foundation Volatility Framework 2.6
.....
0x000000003fce1c70      1      0 R--r-d \Device\HarddiskVolume2\Users\Alissa Simpson\Pictures\Password.png
.....
```
Look at that, an image named Password!!! looks interesting, let's dump it.

```
$ volatility -f MemoryDump_Lab2.raw --profile Win7SP1x64 dumpfiles -Q 0x000000003fce1c70 -D lab2_out/
Volatility Foundation Volatility Framework 2.6.1
DataSectionObject 0x3fce1c70   None   \Device\HarddiskVolume2\Users\Alissa Simpson\Pictures\Password.png
```

[![4](/assets/images/Digital-Fronsic/memlabs/lab2/4.png)](/assets/images/Digital-Fronsic/memlabs/lab2/4.png)

If you look closely at the bottom right, you can spot the password.

Now let's use this password to open the database in KeePass.

[![5](/assets/images/Digital_Forensic/memlabs/lab2/5.png)](/assets/images/Digital_Forensic/memlabs/lab2/5.png) | [![6](/assets/images/Digital_Forensic/memlabs/lab2/6.png)](/assets/images/Digital_Forensic/memlabs/lab2/6.png)

The flag is the copied password.

> #### Flag 2: flag{w0w_th1s_1s_Th3_SeC0nD_ST4g3_!!}

Now let's check chrome process but there is no  built in plugin in volatility for chrome so we need to download this repo ans i will show you how to use it .

Github repo has the plugin we need: [Volatility-Plugins](https://github.com/superponible/volatility-plugins)

```
$ volatility --plugins=plugins/ -f MemoryDump_Lab2.raw --profile Win7SP1x64 chromehistory > chromehistory.txt
```

[![7](/assets/images/Digital-Fronsic/memlabs/lab2/7.jpg)](/assets/images/Digital-Fronsic/memlabs/lab2/7.jpg)

We have mega link, let's open it we found zip file , let's download it .

```
$unzip Important.zip 
Archive:  Important.zip
Password is SHA1(stage-3-FLAG) from Lab-1. Password is in lowercase.
   skipping: Important.png 
```
let's do it .
```
$ echo -n flag{w3ll_3rd_stage_was_easy} | sha1sum 
6045dd90029719a039fd2d2ebcca718439dd100a
```
After unzipping the file, I got this image.

[![8](/assets/images/Digital-Fronsic/memlabs/lab2/8.png)](/assets/images/Digital-Fronsic/memlabs/lab2/8.png)

> #### Flag 3: flag{oK_So_Now_St4g3_3_is_DoNE!!}