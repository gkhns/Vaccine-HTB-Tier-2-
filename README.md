#Linux #FTP #SQL #SUID

**NMAP Scan**

```sh
  nmap -sV -sC -p- 10.129.17.227
  ```

* To see the versions of the services running (-sV)
* To perform a script scan using the default set of scripts (-sC)
* To scan all ports from 1 through 65535 (-p-)

Open Ports

21/tcp open  ftp     vsftpd 3.0.3

22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))

![image](https://user-images.githubusercontent.com/99097743/169863478-0bfbc6c9-0ec3-49bf-a48e-13fdbe28576a.png)

**FTP**

Nmap scan highlights that **anonymous FTP login is allowed**

We are able to download the **backup.zip**

![image](https://user-images.githubusercontent.com/99097743/169863972-2e3d4672-6476-4db9-a294-0e3977a458df.png)

We have a situation here, what is the password of the zip file?

![image](https://user-images.githubusercontent.com/99097743/169865265-8d628052-84c0-4743-880b-7b51031655c9.png)

No worries, John the Ripper to rescue!

https://github.com/openwall/john

John the Ripper jumbo - advanced offline password cracker, which supports hundreds of hash and cipher types, and runs on many operating systems, CPUs, GPUs, and even some FPGAs

Here are the steps:


1. Generate the hash of the zipfile and save it as hash

```sh
zip2john backup.zip > hash
```

2. Use **John** to crack the password using commonly used words (rock.txt). If the password does not show up on the terminal, you can see the password in jonh.pot

```sh
john -w=/usr/share/wordlists/rockyou.txt hash
```

Thanks John! We can now unzip the **backup.zip**

* index.php  --> there is a username:password pair with MD5 hash: You can use one of the free services for Decrypt & Encrypt MD5
* style.css

Using the credentials we can log in to the Car Catalogue

![image](https://user-images.githubusercontent.com/99097743/169875305-9ca24fb3-ecf4-4df9-bf70-96ca635e8de5.png)
