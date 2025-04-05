---
layout: post
title: "Hello World, Hello Chaos!"
date: 2025-03-30 00:42:42 +0800
categories: [🤖 tech, 👋🏻 intro]
tags: [🖼️ jekyll, ⚙️ setup, 👶🏻 beginner]

# img_path: /assets/img/posts/ # optional

pin: true # optional

toc: true # optional

comments: true # optional
---

<iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/4mn5HdatHKN7iFGDes9G8i?utm_source=generator" width="100%" height="152" frameBorder="0" allowfullscreen="" allow="clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>

<br><br>
<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=65&section=body" alt=""/>
</p>

## 👋🏻 Welcome to My Digital Playground! <img src="https://media.giphy.com/media/W5eoZHPpUx9sapR0eu/giphy.gif" width="30px" alt="Git"/>&nbsp;

This is my first post on my new GitHub Pages blog, powered by Jekyll.

Here, I plan to document my adventures (and misadventures) in the world of code, security, privacy, and maybe some V for Vendetta style musings.

<!-- Terminal Container -->
<div class="hacker-terminal">
  <div class="hacker-terminal-header">
    <span class="dot red"></span>
    <span class="dot yellow"></span>
    <span class="dot green"></span>
  </div>
  <div class="hacker-terminal-body">
    <span class="prompt">root@chaos:~#</span>
    <span id="new-typewriter-output"></span><span class="cursor blink">▋</span>
  </div>
</div>


<!-- Basic Styling -->
<style>
.hacker-terminal {
  background-color: #1e1e1e; 
  color: #d4d4d4; 
  font-family: 'Consolas', 'Courier New', monospace; 
  border-radius: 5px;
  padding: 15px 20px;
  margin: 25px 0;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.5);
  position: relative; 
  overflow: hidden;
}

.hacker-terminal-header {
  padding-bottom: 8px;
  margin-bottom: 12px;
}

.hacker-terminal-header .dot {
  display: inline-block;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  margin-right: 5px;
}
.hacker-terminal-header .red { background-color: #ff5f57; }
.hacker-terminal-header .yellow { background-color: #ffbd2e; }
.hacker-terminal-header .green { background-color: #27c93f; }

.hacker-terminal-body {
  white-space: pre-wrap; 
  word-wrap: break-word;
}

.hacker-terminal .prompt {
  color: #569cd6; 
  font-weight: bold;
  margin-right: 8px;
}

.hacker-terminal .cursor {
  display: inline-block;
  background-color: #d4d4d4;
  width: 8px; 
  margin-left: 1px;
  opacity: 1; 
  transition: opacity 0.1s; 
}

.hacker-terminal .cursor.blink {
  animation: blink-animation 1s step-end infinite;
}

@keyframes blink-animation {
  0%, 100% { opacity: 1; } 
  50% { opacity: 0; } 
}

.hacker-terminal .cursor.typing-done {
  animation: none; 
  opacity: 1; 
}
</style>

<!-- JavaScript for Typing Effect -->
<script>
(function() { 

  document.addEventListener('DOMContentLoaded', () => {
    const outputElement = document.getElementById('new-typewriter-output');
    const cursorElement = document.querySelector('.hacker-terminal .cursor'); 
    const textToType = 'echo "Chaos is the new black."'; 
    const typingSpeed = 110; 
    const initialDelay = 600; 
    let charIndex = 0;

    if (!outputElement || !cursorElement) {
      console.error("Error: Typewriter target or cursor element not found!");
      return; 
    }

    function type() {
      if (charIndex < textToType.length) {
        outputElement.textContent += textToType.charAt(charIndex);
        charIndex++;
        setTimeout(type, typingSpeed);
      } else {

        cursorElement.classList.remove('blink');
        cursorElement.classList.add('typing-done'); 
        console.log("Typing effect complete.");
      }
    }

    outputElement.textContent = '';
    cursorElement.classList.add('blink'); 
    cursorElement.classList.remove('typing-done'); 

    setTimeout(type, initialDelay);
  });

})(); 
</script>

*Stay tuned...* or don't. The choice, like the vulnerability, is yours to discover.

<br>
<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=65&section=footer" alt=""/>
</p>