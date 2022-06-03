---
title: '[Root Me] Weak Configuration'
date: 2022-04-26 09:08 PM
categories: [Root-Me, Script]
tags: [easy, challenge, script]
permalink: /:categories/:title
---

We got an access via ssh:

```bash
ssh -p 2222 app-script-ch1@challenge02.root-me.org
```

When we enter with our username and password we see this in the personal directory, a file named readme.md:
```
You have to read the .passwd located in the following PATH : /challenge/app-script/ch1/ch1cracked/
```
{: file="~/readme.md" }

Later that I do an sudo -l to see what we can execute with sudo and with what user.

```console
User app-script-ch1 may run the following commands on challenge02: 

(app-script-ch1-cracked) /bin/cat /challenge/app-script/ch1/notes/*`
```

We see there is a same named user but cracked that can execute cat on `/challenge/app-script/ch1/notes/*`

Then is so easy, we just have to follow this path but replacing the * for `../ch1cracked/.passwd` and we got the flag...
