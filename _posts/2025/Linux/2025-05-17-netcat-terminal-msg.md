---
layout: post
title: "[Linux] Terminal Messaging via netcat - PC & Phone Over LAN"
description: Send messages between your phone and PC like a hacker - with netcat & bash.
date: 2025-05-17 23:31:00 +0800
categories: [🤖 tech, 🐧 Linux]
tags: [🐧 Linux, 🖥️ PC, 📱 Phone, 💬 Terminal Messaging, 🔗 Netcat, 🔐 Local Chat, 🌐 LAN, 🚫 No Internet, 🧠 Bash Scripting, 📝 Logging, 🧪 TCP Experiment]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/terminal_msg.png
---

So, with SFTP, we can easily transfer **files** from phone to PC-fast and smooth.

But what if I just wanna send a quick text to my PC instead?
Like… can I shoot a message straight to the terminal from my phone?

And that got me thinking-

How do hackers talk to each other through terminals, like in those classic hacker movies? 
You know, black screens… glowing green letters… mysterious chats flying across the command line…

Well, that's exactly what we're gonna talk about today. (I use 🐧 Pop!OS, but this works on any Linux distro.)

---

## 🧠 Netcat-based Terminal Messaging (No Permission B.S.)

First things first-make sure you can use Termius (or any other SSH app) on your phone to connect to your PC.

Then we can start the magic work:

> 🟢 **Step 1 (on PC)**

```bash
nc -l -p 8888
```

> 🟢 **Step 2 (on Phone)**

```bash
nc 192.168.1.2 8888
```

⬆️ **Change `192.168.1.2` to your PC private ip.**

Once the connection is established:

* One device listens (`nc -l -p 8888`)
* The other connects (`nc IP PORT`)
* Then? **BOTH CAN TYPE** in the same session 🫠💬

Now they're 💞 bound 💞 in a full-duplex socket stream. You can:

* 📝 Type from PC - appears on Phone
* 📱 Type from Phone - appears on PC

**Same session, no need for separate reverse flow.**

> This only works when **Netcat on both ends supports full-duplex**. **Full-Duplex Ethernet** = **Two-Way Street**, all modern routers support it. 

> Some older nc builds were single-direction only, gotta use **two lanes** for back-and-forth. 👇

```yaml
A: nc -l -p 1234
B: nc A_IP 1234
---
B: nc -l -p 5678
A: nc B_IP 5678

```

---

## Scripts

So let's organize all these features in to `sh` scripts:

| Feature          | Description                                    |
| ---------------- | ---------------------------------------------- |
| 🌈 Colors        | "Phone" = Magenta, "PC" = Blue                 |
| 📝 Logging       | Saved to `~/Documents/Chat/chat_shared.log`    |
| ♾️ Infinite loop | Keeps listening / sending forever              |
| 💬 Beautiful UX  | Human-readable, timestamped, stylized messages |

---

### 💻 `pc-chat.sh`

```bash
#!/bin/bash

LOG="$HOME/Documents/Chat/chat_shared.log"
mkdir -p "$(dirname "$LOG")"

while true; do
  nc -l -p 8888 | while IFS= read -r line; do
    msg="[$(date)] \033[1;35mPhone:\033[0m $line"
    echo -e "$msg" | tee -a "$LOG"
  done
done
```

### 📱 `phone-chat.sh`

```bash
#!/bin/bash

LOG="$HOME/Documents/Chat/chat_shared.log"
mkdir -p "$(dirname "$LOG")"

while true; do
  nc <PC_IP> 8888 | while IFS= read -r line; do
    msg="[$(date)] \033[1;34mPC:\033[0m $line"
    echo -e "$msg" | tee -a "$LOG"
  done
done
```

*   😿 If `8888` not a valid port for you, try change it, check [What Port Range Should You Use](#-what-port-range-should-you-use).  
*   🔧 Replace `<PC_IP>` with your actual private IP (e.g., `192.168.1.2`)  
*   📝 The default log path is `~/Documents/Chat/chat_shared.log` - feel free to change that

---

### 💌 Now Both Devices Will Log To:

```bash
~/Documents/Chat/chat_shared.log
```

Here's a sneak peek of what our terminal love looks like when Netcat is purring and TCP's flowing hot:
(yes... we timestamp everything) 😼

<div class="terminal-window">
  <div class="terminal-header">
    <span class="button red"></span>
    <span class="button yellow"></span>
    <span class="button green"></span>
    <span class="terminal-title">root@chaos:~</span>
  </div>
  <div class="terminal-body">
    <div class="log-entry"><span class="timestamp">[Sat May 17 23:04:13 AWST 2025]</span> <span class="phone">Phone:</span> <span class="message">knocking on your port again</span></div>
    <div class="log-entry"><span class="timestamp">[Sat May 17 23:04:19 AWST 2025]</span> <span class="pc">PC:</span> <span class="message">it's open for you, always</span></div>
    <div class="log-entry"><span class="timestamp">[Sat May 17 23:04:28 AWST 2025]</span> <span class="phone">Phone:</span> <span class="message">damn... that full duplex feels good</span></div>
    <div class="log-entry"><span class="timestamp">[Sat May 17 23:04:34 AWST 2025]</span> <span class="pc">PC:</span> <span class="message">slide that string over, babe</span></div>
    <div class="log-entry"><span class="timestamp">[Sat May 17 23:04:41 PM AWST 2025]</span> <span class="phone">Phone:</span> <span class="message">echo "i want you..." | nc 192.168.1.2 2600</span></div>
    <div class="log-entry"><span class="timestamp">[Sat May 17 23:04:50 AWST 2025]</span> <span class="pc">PC:</span> <span class="message">packet received... loud & clear</span></div>
    <div class="log-entry"><span class="timestamp">[Sat May 17 23:05:01 AWST 2025]</span> <span class="phone">Phone:</span> <span class="message">keep this session alive, don't ctrl+C me tonight</span></div>
    <div class="log-entry"><span class="timestamp">[Sat May 17 23:05:12 AWST 2025]</span> <span class="pc">PC:</span> <span class="message">i'll tee you all night long, sugarbyte</span></div>
  </div>
</div>

<style>
  .terminal-window {
    background-color: #1e1e1e;
    border-radius: 8px;
    overflow: hidden;
    font-family: 'Courier New', monospace;
    color: #cec5b4;
    box-shadow: 0 0 20px rgba(0,0,0,0.7);
    max-width: 750px;
    margin: 2rem auto;
  }

  .terminal-header {
    background-color: #2e2e2e;
    padding: 0.4rem 1rem;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    border-bottom: 1px solid #444;
  }

  .terminal-title {
    margin-left: auto;
    color: #6bc1ff;
    font-weight: bold;
    font-size: 0.9rem;
  }

  .button {
    width: 12px;
    height: 12px;
    border-radius: 50%;
    display: inline-block;
  }

  .red { background-color: #ff5f56; }
  .yellow { background-color: #ffbd2e; }
  .green { background-color: #27c93f; }

  .terminal-body {
    padding: 1rem;
    font-size: 0.95rem;
  }

  .log-entry { margin-bottom: 0.5rem; }
  .timestamp { color: #777; margin-right: 0.5rem; }
  .phone { color: #fa7ea8; font-weight: bold; }
  .pc { color: #6bc1ff; font-weight: bold; }
  .message { color: #cec5b4; }
</style>


## 🧪 Quick Troubleshoot Checklist

If you've used this before and it suddenly breaks, try these steps to see if they help:

### ✅ 1. **Is your PC Still Listening?**

On PC, try in a fresh terminal:

```bash
nc -l -p 8888
```

**Then from the same PC terminal**, in another tab:

```bash
echo "test" | nc 127.0.0.1 8888
```

> 🧪 If this doesn't work locally, `nc` is dead or the port is stuck.
> 💣 Kill any stuck `nc` processes:

```bash
ps aux | grep nc
kill -9 <pid>
```

---

### ✅ 2. **Confirm IPs Are Still Valid**

PC:

```bash
ip a | grep 192
```

→ You should see your private ip like `192.168.1.2`

---

### ✅ 3. **Can Phone Still Ping PC?**

On Phone (via Termius or app):

```bash
ping <PC_IP>
```

✅ = network is fine
❌ = something wrong with Wi-Fi, gateway, or router

> 🧠 If IPs changed (e.g. DHCP refresh), they may no longer match

---

### ✅ 4. **Try Nmap from PC:**

```bash
nmap -Pn <PHONE_IP>
```

Check if **any** port shows up. If not, maybe your phone reconnected to a **guest Wi-Fi**, VPN, or hotspot.

---

### ✅ 5. **Use `netstat` to Check Listeners**

On PC:

```bash
sudo netstat -tulnp | grep 8888
```

If you don't see:

```
tcp   0  0 0.0.0.0:8888   0.0.0.0:*   LISTEN
```

Then PC isn't listening on port 8888 anymore.

---

### ✅ 6. **Try Alternate Port (e.g., 9999)**

Sometimes the port gets "sticky." Try:

PC:

```bash
nc -l -p 9999
```

Phone:

```bash
nc <PC_IP> 9999
```

---

### 🛠️ BONUS - Wipe Netcat Stuck Port

Sometimes a crashed `nc` process holds the port hostage. Run on PC:

```bash
sudo fuser -k 8888/tcp
```

Then try again.

---

### 👀 If You Still Need a Reset:

Reboot both devices' Wi-Fi (or even full reboot).
Then try simple test again:

```bash
# On PC
nc -l -p 8888

# On Phone
nc <PC_IP> 8888
```

Type anything - you should see it instantly.

---

## TL;DR: Try This First

1. Kill all stuck nc:

   ```bash
   pkill nc
   ```

2. Try:

   ```bash
   nc -l -p 9999      # On PC
   nc <PC_IP> 9999   # On Phone
   ```

---

## 💥 Why Did Port Like 8888 Break?

### Possible reasons:

| Reason                           | Explanation                                                          |
| -------------------------------- | -------------------------------------------------------------------- |
| 🔒 **Leftover `nc` process**     | A stuck background listener was still holding `8888` hostage         |
| ❌ **Unclean disconnects**        | When a session is broken forcefully, it can "linger" in OS memory    |
| 🧱 **System firewall / timeout** | Some systems automatically block a port after a ton of reconnections |

> 💡 Even after the script exits, the **port stays in TIME\_WAIT** state for a bit

### 1. 🔁 **TIME\_WAIT ghost state**

Some OSes (especially Linux) treat previously-used ports as "sensitive" if many rapid connects/disconnects occurred.
It'll **let `nc` bind**, but TCP refuses new connections from another device due to **linger settings**.

Try checking with:

```bash
sudo ss -tanp | grep 8888
```

---

### 2. 🔐 **iptables or firewall weirdness**

Something might have marked 8888 as "bad" after repeated scans or failed handshakes.

Check firewall rules:

```bash
sudo iptables -L -n | grep 8888
```

> You might see `DROP` or `REJECT` rules.

---

### 3. 🧠 **Some apps love port 8888**

```bash
> nmap -p 8888 192.168.1.2
Starting Nmap 7.80 ( https://nmap.org ) at 2025-05-17 21:44 AWST
Nmap scan report for PC (192.168.1.2)
Host is up (0.000034s latency).

PORT     STATE  SERVICE
8888/tcp closed sun-answerbook
```

Fun fact:

* `8888` is commonly used by:

  * Jupyter Notebook
  * Some HTTP proxies
  * Sun's `answerbook` service (weird label in nmap 😅)
* Even if not running now, your OS might have network settings baked around it

---

## 🧠 What Port Range Should You Use?

| Range        | Notes                                                     |
| ------------ | --------------------------------------------------------- |
| 1024-49151   | ✅ **Safe to use** - unprivileged ports                    |
| >49152-65535 | 🟡 Dynamic / ephemeral - may be used by OS                |
| <1024        | 🔴 Privileged - needs `sudo` to bind (don't use for chat) |

> ✅ Best range: **1025-49151**
> e.g., `1337`, `2025`, `5454`, `9999`, `42069`... get creative 😉

---

## 💬 Final Thoughts

You don't need Slack, Signal, or servers.  
You just need two devices, netcat, and a love for the terminal 

Whether you're debugging your LAN, leaving secret logs, or flirting through the command line -  
this is messaging in its rawest, sexiest form.

Now go slide that string over, and let your sockets talk 💗

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
