---
title: "Testing Hugo Markup"
date: 2023-05-15T23:58:09-07:00
draft: false
---
It's a particle! In a box!
<!--more-->

<canvas id="particlecanvas" width="300" height="100"></canvas>
<script>
const c = document.getElementById("particlecanvas");
const ctx = c.getContext("2d")
let pos = 0;
let v = 1;
function step() {
    ctx.clearRect(0,0,300,100);
    ctx.strokeRect(0,0,300,100);
    ctx.fillRect(0+pos, 40, 10, 10);
    if (pos >= 290) {
        v *= -1;
    }
    if (pos <= 0 && v < 0) {
        v *= -1;
    }
    pos += v;
}
window.setInterval(step, 10);
</script>

Just testing out how to include little javascript snippets in Hugo markdown.