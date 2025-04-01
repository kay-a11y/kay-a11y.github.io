---
layout: post
title: "Hello World, Hello Chaos!"
date: 2025-03-30 00:42:42 +0800
categories: [tech, intro]
tags: [jekyll, setup, beginner]

# img_path: /assets/img/posts/ # optional

pin: true # optional

toc: true # optional

comments: true # optional
---

<iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/4mn5HdatHKN7iFGDes9G8i?utm_source=generator" width="100%" height="152" frameBorder="0" allowfullscreen="" allow="clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>

<br><br>
<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=65&section=body"/>
</p>

## Welcome to My Digital Playground! <img src="https://media.giphy.com/media/W5eoZHPpUx9sapR0eu/giphy.gif" width="30px" alt="Git"/>&nbsp;

This is my first post on my new GitHub Pages blog, powered by Jekyll.

Here, I plan to document my adventures (and misadventures) in the world of code, security, privacy, and maybe some V for Vendetta style musings.

<!-- Terminal Container -->
<div class="terminal-container">
  <div class="terminal-header">
    <span></span> <!-- Red -->
    <span></span> <!-- Yellow -->
    <span></span> <!-- Green -->
  </div>
  <div class="terminal-body">
    <span class="prompt">user@chaos:~$</span> <span id="typed-output"></span><span class="cursor">▋</span>
  </div>
</div>

<!-- Basic Styling -->
<style>
.terminal-container {
  background-color: #282c34; /* Dark background */
  color: #abb2bf; /* Light grey text */
  font-family: 'Consolas', 'Monaco', monospace; /* Monospace font */
  border-radius: 6px;
  padding: 15px;
  margin: 20px 0; /* Add some space around it */
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
  overflow: hidden; /* Hide anything that might overflow */
}

.terminal-header {
  padding-bottom: 10px;
  border-bottom: 1px solid #444;
  margin-bottom: 10px;
}

.terminal-header span {
  display: inline-block;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  margin-right: 6px;
}
.terminal-header span:nth-child(1) { background-color: #ff5f56; } /* Red */
.terminal-header span:nth-child(2) { background-color: #ffbd2e; } /* Yellow */
.terminal-header span:nth-child(3) { background-color: #27c93f; } /* Green */

.terminal-body {
  /* Ensure text wrapping behaves correctly */
  white-space: pre-wrap;
  word-wrap: break-word;
}

.prompt {
  color: #61afef; /* Blue prompt color */
  font-weight: bold;
  margin-right: 5px;
}

/* Blinking cursor effect */
.cursor {
  display: inline-block; /* Needed for animation */
  background-color: #abb2bf; /* Cursor color same as text */
  width: 8px; /* Width of the cursor */
  margin-left: 2px; /* Space before cursor */
  animation: blink 1s step-end infinite;
}

.cursor.stopped {
  animation: none; /* Remove the animation */
  /* Optional: Ensure the cursor remains visible as a solid block */
  background-color: #abb2bf;
}

@keyframes blink {
  from, to { background-color: transparent }
  50% { background-color: #abb2bf; } /* Visible state */
}
</style>

<!-- JavaScript for Typing Effect -->
<script>
  const textToType = 'print("Chaos is the new black.")';
  const outputElement = document.getElementById('typed-output');
  const typingSpeed = 100; // Milliseconds per character (adjust speed here)
  let i = 0;

  function typeWriter() {
    if (i < textToType.length) {
      outputElement.textContent += textToType.charAt(i);
      i++;
      setTimeout(typeWriter, typingSpeed);
    } else {
      // Typing finished: Add 'stopped' class to the cursor
      const cursorElement = document.querySelector('.terminal-body .cursor');
      if (cursorElement) {
        cursorElement.classList.add('stopped');
      }
    }
  }

  // Ensure the script runs after the element exists
  // If you put this script at the end of your body, DOMContentLoaded might not be necessary
  // But it's safer practice
  document.addEventListener('DOMContentLoaded', (event) => {
    // Clear any previous text (if needed)
    outputElement.textContent = '';
    // Start typing
    setTimeout(typeWriter, 500); // Add a small delay before starting
  });
</script>

*Stay tuned...* or don't. The choice, like the vulnerability, is yours to discover.

<br>
<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=65&section=footer"/>
</p>