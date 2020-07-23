+++
title = "My Pathbot experiment"
date = 2019-07-09
description = "A little game base on GitHub's Pathbot experiment, where you have to escape a maze."

[extra]
extract_from = "description"
+++

<div style="float: right; display: flex">
  <img src="rust-logo-blk-72px.png" style="object-fit: contain"/>
  <img src="web-assembly-logo-128px.png" style="object-fit: contain"/>
</div>

A little Pathbot experiment.

[Source code](https://github.com/totorigolo/pathbot)

<br style="clear: both" />

**EDIT 2020-07-23**: Last year, approx. June 2019, GitHub introduced the [Noops
Challenge][noops-challenge], with a simple concept:

> Love a challenge? The Noops are inexplicable machines that don’t do anything
> at all. Pick one, make it do something, and have fun with code along the way.

Basically, they created bots with simple APIs, especially designed to offer a
wide range of hacks based upon them. I got inspired by [Pathbot][pathbot], and
took the opportunity to create a (small-but-) real project with [Yew][yew], a
Rust framework inspired by React and Elm that compiles to WebAssembly.

The code is open source, you can find the link above. Below is how this is
integrated on this website.

[noops-challenge]: https://noopschallenge.com/
[pathbot]: https://noopschallenge.com/challenges/pathbot
[yew]: https://yew.rs/

```html
<div id="pathbot-root"></div>
<script defer src="./pathbot.js"></script>
```

---

<div id="pathbot-root"></div>
<script defer src="./pathbot.js"></script>

