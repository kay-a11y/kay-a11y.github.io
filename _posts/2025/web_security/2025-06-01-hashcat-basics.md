---
layout: post
title: "Hashcat Basics: Cracking Passwords with Masks, Modes & Benchmarks"
description: "Learn how to use Hashcat to crack passwords, test password strength, and benchmark your GPU using real masks, hash modes, and strategies - all in a hands-on, ethical hacking walkthrough."
date: 2025-06-01 21:23:00 +0800
categories: [🤖 tech, 🔒 Web Security]
tags: [🐧 Linux, 💻 Networking, 🐾 Penetration, 🐾 Hacker Basics, 🔒 Web Security, 🔒 Privacy, 🧪 Cryptography, 😼 Hashcat, 🧔 JohnTheRipper, 🔒 Password Cracking, 🖥️ CLI, 🐾 Brute Force]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: 
---

> **Disclaimer:**  
> This post is shared **strictly for educational and ethical purposes**.  
> if you're gonna crack, do it with consent and curiosity.

My curiosity was sparked by this awesome video - and guys, you *gotta* check it out:

Yup, Kevin walks us through how to use **Hashcat** in this demo.

<p align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/K-96JmC2AkE?si=4cNp-e7IUgr_F7FO" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</p>

Let's learn how to use **Hashcat** to test how vulnerable *your* password really is.

---

## 💡 Step-by-Step Hashcat Setup

### 🔧 1. **Install dependencies**

```bash
sudo apt update
sudo apt install -y hashcat
```

Confirm:

```bash
hashcat -I
```

✅ You should see your GPU listed

You might got a **little warning** about `Failed to initialize NVIDIA RTC library`. It's not a showstopper at all - just a *missed bonus*, and here's what it means:

---

### 🚨 What's This RTC Thing?

RTC = **Ray Tracing Compilation** Library
It's a part of NVIDIA's CUDA toolkit that allows JIT (just-in-time) compiling of optimized GPU code in CUDA 11.0+.

Hashcat tries to use this for:

* Faster runtime compiling of kernels
* Optimized CUDA-specific enhancements

If it fails?
→ **No biggie.** It just falls back to **OpenCL**.

Actually you **don't have to fix it**, but if you want that *maximum GPU phreak mode™*, then let's get you the CUDA toolkit.

```bash
sudo apt install nvidia-cuda-toolkit
```

> This installs the **CUDA compiler & libraries**, including the missing RTC lib

Then restart your system or reload drivers:

```bash
sudo reboot
```

After reboot, run:

```bash
hashcat -I
```

And that RTC message disappears.

---

### ➡️ Use `passlib` - a Python lib that *still supports* NTLM hashing

```bash
pip install passlib
```

Then run:

```bash
python3 -c "from passlib.hash import nthash; print(nthash.hash('wire2600'))"
```

> 🔓 Output:

```
4eb13fcf15bd6e0fde1d1e768c1cf32b
```

That's your **NTLM hash**.

Put it in `hash.txt`:

```bash
echo 4eb13fcf15bd6e0fde1d1e768c1cf32b > hash.txt
```

And you're back on the cracking track.

#### ❓ Why NTLM?

**NTLM** isn't just for Windows hashes!  
But:

* It's one of the *fastest* and *simplest* hashes to generate and crack (perfect for demos, CTF, and speed testing).
* **Hashcat** supports a *ton* of hash types (`hashcat --help | grep -i 'mode'`), but NTLM (`-m 1000`) is always an easy, standardized "playground" hash.
* In real pentests, you'd target whatever you extract (shadow files = `-m 1800` for Linux, etc).
* So **NTLM is just a teaching/demo classic** - not a "must" for Linux.

#### 🐱 **Hashcat Works With All Hash Types**

Just pick the mode:

* **NTLM**: `-m 1000`
* **SHA512crypt**: `-m 1800`
* **MD5**: `-m 0`
* **SHA1**: `-m 100`
* **LM**: `-m 3000`
* **bcrypt**: `-m 3200`

(You can see ALL with: `hashcat --help`)

| Scenario                   | What to Use          | Hashcat Mode |
| -------------------------- | -------------------- | ------------ |
| Windows login hash         | NTLM                 | -m 1000      |
| Linux `/etc/shadow` hash(Linux login)   | SHA512crypt          | -m 1800      |
| Just want a demo/test hash | NTLM (super fast)    | -m 1000      |
| Any other hash             | See `hashcat --help` | Varies       |

---

## 🧨 Cracking Track

Now `hash.txt` contains the target hash.
Hashcat will try millions of password guesses to reverse this.

We know:

* Password is 8 chars: 4 lowercase + 4 digits (`wire2600`)
* So we can use a **mask attack**, not full brute-force
  → way faster

---

### 🎭 Run Hashcat (With Mask)

```bash
hashcat -m 1000 -a 3 hash.txt "?l?l?l?l?d?d?d?d"
```

#### 📈 Breakdown:

| Flag               | Meaning                             |
| ------------------ | ----------------------------------- |
| `-m 1000`          | NTLM hash mode                      |
| `-a 3`             | Brute-force (mask attack)           |
| `hash.txt`         | File with the hash to crack         |
| `?l?l?l?l?d?d?d?d` | Mask: 4 lowercase letters, 4 digits |

---

### 🧝🏻 Watch the Magic

Hashcat will show you:

* How many guesses per second
* Which GPU it's using
* Estimated time
* Found password (recovered 1/1)

I am using 3060 Ti, this cost only 3 sec!

```txt
4eb13fcf15bd6e0fde1d1e768c1cf32b:wire2600

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1000 (NTLM)
Hash.Target......: 4eb13fcf15bd6e0fde1d1e768c1cf32b
Time.Started.....: Sun Jun  1 21:51:14 2025 (0 secs)
Time.Estimated...: Sun Jun  1 21:51:14 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Mask.......: ?l?l?l?l?d?d?d?d [8]
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 13517.3 MH/s (1.00ms) @ Accel:64 Loops:128 Thr:512 Vec:1
Recovered........: 1/1 (100.00%) Digests
Progress.........: 632320000/4569760000 (13.84%)
Rejected.........: 0/632320000 (0.00%)
Restore.Point....: 0/260000 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:4736-4864 Iteration:0-128
Candidate.Engine.: Device Generator
Candidates.#1....: anee1999 -> mupq6492
Hardware.Mon.#1..: Temp: 39c Fan:  0% Util:  0% Core:1740MHz Mem:6801MHz Bus:16

Started: Sun Jun  1 21:51:12 2025
Stopped: Sun Jun  1 21:51:15 2025
```

---

### 🧑🏻‍🎤 rockyou

To load `rockyou.txt`:

```bash
wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
```

Then:

```bash
hashcat -m 1000 -a 0 hash.txt rockyou.txt
```

---

## 🎭 What Is a Mask in Hashcat?

A **mask** is a way to tell Hashcat:

> "Hey bro, try **passwords that follow this pattern**."

It's **not** trying every possible combo of every character - just what *you* tell it to. That's why it's **lightning fast** and **laser-accurate** when you have even a *vague idea* of the password's structure.

---

### 🎨 Writing Masks: The Symbol Cheatsheet

| Symbol | Meaning                        | Example               |
| ------ | ------------------------------ | --------------------- |
| `?l`   | Lowercase letter (a-z)         | `a`, `f`, `z`         |
| `?u`   | Uppercase letter (A-Z)         | `D`, `X`              |
| `?d`   | Digit (0-9)                    | `1`, `8`              |
| `?s`   | Special char (`!@#$%^&*`, etc) | `@`, `!`              |
| `?a`   | [All printable ASCII](https://www.ascii-code.com/characters/printable-characters){:target="_blank"}            | `a`, `9`, `@`         |
| `?b`   | All 8-bit bytes (brutal mode)  | non-printables too    |
| `?h`   | Lowercase hex (`0-9a-f`)       | `c`, `4`              |
| `?H`   | Uppercase hex (`0-9A-F`)       | `E`, `A`              |

---

### 💡 Example Masks

| Mask               | Cracks...                     |
| ------------------ | ----------------------------- |
| `?l?l?l?l?d?d?d?d` | Like `wire2600`               |
| `?u?u?u?u?d?d`     | Like `ABCD12`                 |
| `?l?l?d?d?s?s`     | Like `ab12!!`                 |
| `?d?d?d?d`         | 4-digit PIN                   |
| `?a?a?a?a?a?a`     | Any 6-char ASCII password     |
| `password?d?d`     | Hybrid mask like `password12` |

> You can even use files or generate custom mask sets

---

## 🤔 What If We Don't Know the Password?

What we just did is amazing for:

* **Testing your own password's strength**
* **Benchmarking**
* **Targeted cracking when you know part of the format**

But if you **don't know the password at all**, you have **4 attack strategies** to choose from:

---

### 💃 Hashcat Attack Modes (aka Choose Your Fighter)

| Mode | Name                 | Command Flag | When to Use                                           |
| ---- | -------------------- | ------------ | ----------------------------------------------------- |
| 0    | **Dictionary**       | `-a 0`       | You think the password is in a wordlist               |
| 1    | **Combinator**       | `-a 1`       | Combine two lists (like `first` + `last`)             |
| 3    | **Mask**             | `-a 3`       | You know part of the pattern (like `abcd1234`)        |
| 6    | **Hybrid Word+Mask** | `-a 6`       | Add mask to end of each word (like `rockyou` + `123`) |
| 7    | **Hybrid Mask+Word** | `-a 7`       | Add word to end of mask (like `123` + `password`)     |

We used **mode 3** just now.

---

#### ✨ Examples

##### 🧱 1. **Wordlist Attack** (`-a 0`)

```bash
hashcat -m 1000 -a 0 hash.txt rockyou.txt
```

> Try every word in the `rockyou.txt` list as password

When you're using a **dictionary attack** (`-a 0`) and the password isn't in the wordlist:

> **Hashcat will finish** and show:

```
Status.........: Exhausted
Recovered......: 0/1 (0.00%)
```

That just means:

> "Bruhh... I tried everything you gave me... and it wasn't there."

##### 🧬 2. **Hybrid Attack** (`-a 6`)

There are two kinds:

| Type                     | Flag   | Pattern                           |
| ------------------------ | ------ | --------------------------------- |
| Wordlist + Mask (suffix) | `-a 6` | `rockyou.txt` + `123`, `!@#`, etc |
| Mask + Wordlist (prefix) | `-a 7` | `123`, `admin` + `rockyou.txt`    |

```bash
hashcat -m 1000 -a 6 hash.txt rockyou.txt '?d?d'
```

> Like guessing `password01`, `letmein99`, etc.

##### ⚔️ 3. **Brute Force (Mask)** when you know structure

```bash
hashcat -m 1000 -a 3 hash.txt '?u?l?l?l?l?d?d'
```

---

### ❓ What If We Don't Know Password Length?

Let's say you wanna brute-force ASCII passwords of *unknown length*…

---

#### 🧨 Use Mask + Increment Mode

```bash
hashcat -m 1000 -a 3 hash.txt '?a?a?a?a?a?a' --increment
```

This will try:

* 1-char
* 2-char
* ...
* Up to 6-char ASCII passwords.

You can control length:

```bash
--increment-min 4 --increment-max 8
```

Full example:

```bash
hashcat -m 1000 -a 3 hash.txt '?a?a?a?a?a?a?a?a' --increment --increment-min 4 --increment-max 8
```

This tries **all printable ASCII** passwords from 4 to 8 characters.

---

#### ⚠️ Warning

* `?a` = 95 characters (a-z, A-Z, 0-9, symbols)
* `?a?a?a?a?a?a?a?a` = **95^8 ≈ 6.6 quadrillion guesses**
* Even 40s/50s cards will take a *long ass time* without some kind of **hint**

---

##### ⌛ How long will you take based on your mask?

Let's figure out the unit first:

| Unit           | Meaning                                 |
| -------------- | --------------------------------------- |
| `MH/s`         | **MegaHashes per second**               |
| `1 MH/s`       | = **1,000,000 (1 million)** guesses/sec |
| `11146.1 MH/s` | = **11,146,100,000 guesses/sec**        |

> This is how many password **candidates** your GPU is testing against the target hash **every second**.

Say your speed is `Speed: 56198.2 MH/s` = 56,198,200,000 guesses/sec  
Say you are cracking with `hashcat -m 1000 -a 3 hash.txt '?a?a?a?a?a?a?a?a' --increment --increment-min 4 --increment-max 8`

That's

```
(95⁴ + 95⁵ + 95⁶ + 95⁷ + 95⁸) ÷ 56198200000 ÷ 3600
= 33.14 hours
```

That'll have your sexy card grinding non-stop for 33.14 hours straight!  
(*GPU's basically clocked in. Where's the ETH?* 🤨)

---

##### 🔥 Real Statistics of 3060 Ti

| Mask Length | Mask               | Speed (MH/s) | Temp | Util | Time     |
| ----------- | ------------------ | ------------ | ---- | ---- | ------------- |
| 4 chars     | `?a?a?a?a`         | 2169.1       | 47°C | 5%   | 2 secs        |
| 5 chars     | `?a?a?a?a?a`       | 2581.6       | 53°C | 98%  | 4 secs        |
| 6 chars     | `?a?a?a?a?a?a`     | 2092.4       | 57°C | 99%  | 5 min         |
| 7 chars     | `?a?a?a?a?a?a?a`   | 1930.1       | 54°C | 98%  | 10 hrs 2 mins (Estimated) |
| 8 chars     | `?a?a?a?a?a?a?a?a` | 3451.6       | 58°C | 99%  | \~22 days (Estimated)  |

---

##### 🧠 How to Benchmark All Modes Speed?

To see how fast your card cracks different algorithms:

```bash
hashcat -b
```

This runs a **benchmark** on all supported hash types and shows:

* `Speed.#1` per algorithm
* Kernel used (optimized or not)
* Time per batch

Output will look like:

```
-------------------
* Hash-Mode 0 (MD5)
-------------------
Speed.#1.........: 34264.1 MH/s (73.85ms) @ Accel:2048 Loops:1024 Thr:32 Vec:8
----------------------
* Hash-Mode 100 (SHA1)
----------------------
Speed.#1.........: 10646.1 MH/s (59.36ms) @ Accel:256 Loops:256 Thr:256 Vec:1
---------------------------
* Hash-Mode 1400 (SHA2-256)
---------------------------
Speed.#1.........:  4685.5 MH/s (67.73ms) @ Accel:8 Loops:1024 Thr:1024 Vec:1
```

This helps you compare:

* Fast hashes (MD5, NTLM, SHA1)
* Slow, secure ones (bcrypt, SHA512crypt, PBKDF2, etc.)

---

## 🥊 **John the Ripper vs Hashcat: Battle of the Cracking Titans**

| Feature             | 🧔 John the Ripper (JtR)                       | 😼 Hashcat                              |
| ------------------- | ---------------------------------------------- | --------------------------------------- |
| Engine           | CPU-based (but has GPU version: *Jumbo* build) | Native GPU-based                        |
| Speed            | Slower on CPU, decent on GPU Jumbo             |  King of speed (especially GPU)       |
| Smartness        | Rule-based + hybrid + incremental              | Rule-based, mask, hybrid, brute, PRINCE |
| Out of the box   | Simpler to start, can autodetect hash types    | Requires specifying hash modes          |
| Setup Difficulty | Easier for beginners                           | Slightly steeper learning curve         |
| Wordlist Support | Yes (e.g., rockyou.txt)                        | Yes (with crazy fast rule mutations)    |
| Best Use Case    | Quick test on CPU, CTFs, offline boxes         | Industrial-grade cracking (real ops)    |

---

### 🔥 When to Use What?

| Situation                           | Go With…   |
| ----------------------------------- | ---------- |
| You're on Linux terminal, no GPU    | 🧔 John    |
| You want max performance            | 🐱 Hashcat |
| Doing CTFs or quick offline checks  | 🧔 John    |
| Full-blown audit with big hashlists | 🐱 Hashcat |

---

> 🧠 Ready to flex those GPU muscles harder?  
> 👉🏻 Check out Part 2: [Crack That ZIP - Using John + Hashcat (`No hashes loaded Error` Fix Included)](https://kay-a11y.github.io/posts/hashcat-john-zip/)

<div class="donation-box" style="position: relative;">
  <p class="donation-text">💖 Support me with crypto or PayPal! 💘</p>
  <p><strong>💵 USDT (TRC20):</strong><br>TJCANuMYSdgLKRKnpCtscXrS5NgDbBAvF9</p>
  <p><strong>🟠 Bitcoin (BTC):</strong><br>bc1qrc9vhrrhnc9v9s9q9rjn24aj608j44p5hzsxft</p>
  <p>Or support me on Ko-fi:</p>
  
  <div class="img-container" style="position: relative; display: inline-block;">
    <img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png"
         alt="Support me on Ko-fi"
         width="150"
         loading="lazy">    
    <div onclick="window.open('https://ko-fi.com/kikisec', '_blank')" 
         style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: transparent; cursor: pointer;">
    </div>
  </div>

  <p class="donation-note">Any amount helps me continue creating content 💬💻</p>
</div>
