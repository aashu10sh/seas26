---
# try also 'default' to start simple
theme: apple-basic
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# some information about your slides (markdown enabled)
title:  Software Engineering at Scale
info: |
  ## Software Engineering at Scale for DMT Interns and more!

# apply UnoCSS classes to the current slide
# class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable Comark Syntax: https://comark.dev/syntax/markdown
comark: true
# duration of the presentation
duration: 60min
---

We are starting in 5 mins
```js {monaco-run}
let totalSeconds = 5 * 60; 
const timer = setInterval(() => {
  console.clear();
  const minutes = Math.floor(totalSeconds / 60);
  const seconds = totalSeconds % 60;

  const formattedSeconds = seconds < 10 ? '0' + seconds : seconds;
  console.log(`Starting in: ${minutes}:${formattedSeconds}`);

  if (totalSeconds <= 0) {
    clearInterval(timer);
    console.log("Time's up!");
  } else {
    totalSeconds--;
  }
}, 1000);

```

---
layout: intro
---
# Software Engineering at Scale.

<div class="absolute bottom-10">
    <span class="font-500">
        aashutosh p
    </span>
</div>

---
layout: image-right
image: https://github.com/aashu10sh/seas26/blob/main/aashutosh.jpg?raw=true
---
Aashutosh Pudasaini

- In love with computers since the 8th grade
- Software Engineer at Leapfrog Technology - writing go on AWS for a british neobank
- distributed systems nerd ( future goal )
- football
- philosophy
- open source

---
# Day 1
src: days/day1.md
---

---
# Day 2
src: days/day2.md
---

---
# Day 3
src: days/day3.md
---

---
# Day 4
src: days/day4.md
---
