---
layout: post
title: "Mute the annoying bell sound in Linux"
description:  Murder that terminal beep once and for all - with one tiny config spell. 💀
date: 2025-04-20 19:30:00 +0800
categories: [🤖 tech, 🐧 Linux]
tags: [🐧 Ubuntu, 🧡 Ubuntu, ⚙️ WSL2, 🪟 Windows11, 👩🏻‍💻 glow-up]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/Linux_mute.png
---

## That Annoying Backspace Beep

You know the one. The **terminal beep** — that ear-piercing "BEEP!" that hits you like a sudden betrayal when you backspace one character too far. 😓  
It's called the **“bell”**, but honestly it sounds more like a horror jump-scare.

Lucky for us, there’s a clean and silent way to shut it up. Permanently. 🔪

### 🔇 Disable the Bell via `~/.inputrc`

This method works like real magic, even in WSL2 or pure Ubuntu setups. Here's how to exorcise that beep:

1. Open your `inputrc` file:

   ```bash
   nano ~/.inputrc
   ```

2. Add this line to the bottom:

   ```bash
   set bell-style none
   ```

3. Save and exit.  
   - `Ctrl + O` → Enter to save  
   - `Ctrl + X` to exit  

4. Reload the config:

   ```bash
   bind -f ~/.inputrc
   ```

And that’s it. The beast is dead. 🪦

> 🧠 *Note:* This will mute the bell sound for Bash sessions. It works perfectly on WSL2, Ubuntu, and other Debian-based distros. For Zsh users, you might want to do something similar in your `.zshrc`.

**💬 Got your terminal to finally shut up? Leave a comment below and brag a little. You earned it. 😏**
