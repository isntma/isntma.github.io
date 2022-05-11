---
title: Linux Enumeration Guide 
layout: post
date: 2022-05-11 9:31:32
permalink: blog/:categories/:title
categories: notes forensics
---

## Initial Information Gathering

### Basic Information

You can start **extracting some basic information**:

```bash
date #Date and time
uname -a #OS info
ifconfig -a || ip a #Network interfaces 
ps -ef #Running processes
netstat -anp #Proccess and ports
lsof -V #Open files
netstat -rn; route #Routing table
df; mount #Free space and mounted devices
free #Meam and swap space
w #Who is connected
last -Faiwx #Logins
lsmod #What is loaded
```

#### Suspicious information

While obtaining the basic information you should check for weird things like:

* **root processes** usually run with low PIDS, so if you find a root process with a big PID you may suspect
* Check **registered logins** of users without a shell inside `/etc/passwd`
* Check for **password hashes** inside `/etc/shadow` for users without a shell

### Disk Imaging

#### Taking an image of the disk

It's important to note that **before connecting to your computer anything related to the case**, you need to be sure that it's going to be **mounted as read only** to avoid modifying the any information.

```bash
#Create a raw copy of the disk
$ dd if=<subject device> of=<image file> bs=512
#Raw copy with hashes along the way (more secure it checks hashes while it's copying the data)
$ dcfldd if=<subject device> of=<image file> bs=512 hash=<algorithm> hashwindow=<chunk size> hashlog=<hash file>
```

### Disk Image pre-analysis

Imaging that you receive a disk image with no more data.

```bash
#Find that it's actually a disk imageusing "file" command
$ file disk.img 
#Check which type of disk image it's
$ img_stat -t evidence.img 
#You can list supported types with
$ img_stat -i list
#Data of the image
$ fsstat -i raw -f ext4 disk.img 
#ls inside the image
$ fls -i raw -f ext4 disk.img
#ls inside folder
$ fls -i raw -f ext4 disk.img 12
#cat file inside image
$ icat -i raw -f ext4 disk.img 16
```

## Search for known Malware

### Modified System Files

Some Linux systems have a feature to **verify the integrity of many installed components**, here some ways to do it.

```bash
#RedHat
$ rpm -Va
#Debian
$ dpkg --verify
$ debsums | grep -v "OK$"
#Arch
$ paccheck (part of pacutils)
```

## Search installed programs

### Package Manager

Different ways to check installed packages:

```bash
#Debian
$ cat /var/lib/dpkg/status | grep -E "Package:|Status:"
$ cat /var/log/dpkg.log | grep installed
#RedHat
$ rpm -qa --root=/ mntpath/var/lib/rpm
#Arch
$ pacman -Q
```

### Other

**Not all installed programs will be listed by the above commands**. Locations such as _**/usr/local**_ and _**/opt**_ may reveal other apps that have been compiled and installed from source code.

```bash
ls /opt /usr/local
```
Another good idea is to **check** the **common folders** inside **$PATH** for **binaries not related** to **installed packages:**

```bash
#Debian
find /sbin/ -exec dpkg -S {} \; | grep "no path found"
#RedHat
find /sbin/ –exec rpm -qf {} \; | grep "is not"
```

## Inspect AutoStart locations

### Scheduled Tasks

```bash
cat /var/spool/cron/crontabs/*  \
/var/spool/cron/atjobs \
/var/spool/anacron \
/etc/cron* \
/etc/at* \
/etc/anacrontab \
/etc/incron.d/* \
/var/spool/incron/* \
```

### Other AutoStart Locations

There are several configuration files that Linux uses to automatically launch an executable when a user logs into the system that may contain traces of malware.

* _**/etc/profile.d/\***_ , _**/etc/profile**_ , _**/etc/bash.bashrc**_ are executed when any user account logs in.
* _**∼/.bashrc**_ , _**∼/.bash\_profile**_ , _**\~/.profile**_ , _**∼/.config/autostart**_ are executed when the specific user logs in.
* _**/etc/rc.local**_ It is traditionally executed after all the normal system services are started, at the end of the process of switching to a multiuser runlevel.

## Examine Logs

Look in all available log files on the compromised system for traces of malicious execution and associated activities such as creation of a new service.

### Pure Logs

**Logon** events recorded in the system and security logs, including logons via the network, can reveal that **malware** or an **intruder gained access** to a compromised system via a given account at a specific time.\
Interesting system logons:

* **/var/log/syslog** (debian) or **/var/log/messages** (Redhat)
  * Shows general messages and info regarding the system. 
* **/var/log/auth.log** (debian) or **/var/log/secure** (Redhat)
  * Keep authentication logs for both successful or failed logins, and authentication processes.
  * `cat /var/log/auth.log | grep -iE "session opened for|accepted password|new session|not in sudoers"`
* **/var/log/boot.log**: start-up messages and boot info.
* **/var/log/maillog** or **var/log/mail.log:** is for mail server logs.
* **/var/log/kern.log**: keeps in Kernel logs and warning info.
* **/var/log/dmesg**: a repository for device driver messages. Use **dmesg** to see messages in this file.
* **/var/log/faillog:** records info on failed logins. Handy for examining potential security breaches like login credential hacks and brute-force attacks.
* **/var/log/cron**: keeps a record of cron jobs. Like when the cron daemon started a job.
* **/var/log/daemon.log:** keeps track of running background services.
* **/var/log/btmp**: keeps a note of all failed login attempts.
* **/var/log/httpd/**: a directory containing error\_log and access\_log files of the Apache httpd daemon. 
* **/var/log/mysqld.log** or **/var/log/mysql.log** : MySQL log file that records every debug, failure and success message, including starting, stopping and restarting of MySQL daemon mysqld.
* **/var/log/xferlog**: keeps FTP file transfer sessions. 
* **/var/log/\*** : You should always check for unexpected logs in this directory

### Command History

Many Linux systems are configured to maintain a command history for each user account:

* \~/.bash\_history
* \~/.history
* \~/.\*\_history

### Logins

Using the command `last -Faiwx` it's possible to get the list of users that have logged in.\
It's recommended to check if those logins make sense:

* Any unknown user?
* Any user that shouldn't have a shell has logged in?

This is important as **attackers** some times may copy `/bin/bash` inside `/bin/false` so users like **lightdm** may be **able to login**.

### Application Traces

* **SSH**: Connections to systems made using SSH to and from a compromised system result in entries being made in files for each user account (_**∼/.ssh/authorized\_keys**_ and _**∼/.ssh/known\_keys**_). These entries can reveal the hostname or IP address of the remote hosts.
* **Gnome Desktop**: User accounts may have a _**∼/.recently-used.xbel**_ file that contains information about files that were recently accessed using applications running in the Gnome desktop.
* **VIM**: User accounts may have a _**∼/.viminfo**_ file that contains details about the use of VIM, including search string history and paths to files that were opened using vim.
* **Office**: Recent files.
* **MySQL**: User accounts may have a _**∼/.mysql\_history**_ file that contains queries executed using MySQL.
* **Less**: User accounts may have a _**∼/.lesshst**_ file that contains details about the use of less, including search string history and shell commands executed via less

### USB Logs

[**usbrip**](https://github.com/snovvcrash/usbrip) is a small piece of software written in pure Python 3 which parses Linux log files (`/var/log/syslog*` or `/var/log/messages*` depending on the distro) for constructing USB event history tables.

It is interesting to **know all the USBs that have been used** and it will be more useful if you have an authorized list of USB to find "violation events".

### Installation

```
pip3 install usbrip
usbrip ids download #Download database
```

### Examples

```
#Get USB history of your curent linux machine
$ usbrip events history
#Search by pid OR vid OR user
$ usbrip events history --pid 0002 --vid 0e0f --user kali 
#Search for vid and/or pid
$ usbrip ids download #Downlaod database
#Search for pid AND vid
$ usbrip ids search --pid 0002 --vid 0e0f
```

## Examine File System

File system data structures can provide substantial amounts of **information** related to a **malware** incident, including the **timing** of events and the actual **content** of **malware**.\
To deal with such anti-forensic techniques, it is necessary to pay **careful attention to time line analysis** of file system date-time stamps and to files stored in common locations where malware might be found.

* Using **autopsy** you can see the timeline of events that may be useful to discover suspicions activity. You can also use the `mactime` feature from **Sleuth Kit** directly.
* Check for **unexpected scripts** inside **$PATH**.
* Files in `/dev` use to be special files.
* Look for unusual or **hidden files** and **directories**, like “.. ” or “..^G ”.
* setuid copies of /bin/bash on the system `find / -user root -perm -04000 –print`
* Also check directories like _/bin_ or _/sbin_ as the **modified and/or changed time** of new or modified files me be interesting.
* It's interesting to see directories **sorted by creation date** instead alphabetically to see which files/folders are more recent and inodes.
  * You can check the most recent files of a folder using\
  `ls -laR --sort=time /bin`.
  * You can check the inodes of the files inside a folder using\
  `ls -lai /bin | sort -n`
