---
layout: post
title: "[Linux] Spoof Your MAC Address in Linux"
description: 
date: 2025-05-20 20:34:00 +0800
categories: [🤖 tech, 🔒 Web Security]
tags: [🛡️ MAC Address, 🧙‍♀️ Spoofing, 🐧 Linux, 💻 Networking, 🐾 Hacker Basics, 📡 DHCP, 🛡️ IP Refresh]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/spoof_mac.png
---

<iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/1O1LkHTi3Lep9OHE8BvOVe?utm_source=generator" width="100%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>

## 🌟 Quick steps(temporary):

```bash
sudo ip link set dev INTERFACE down
sudo ip link set dev INTERFACE address NEW_MAC_ADDRESS
sudo ip link set dev INTERFACE up
```

---

### 🌸 Example:

Suppose your network card (Wi-Fi or Ethernet) is named `enp2s0` or `wlan0` or something else (we'll check it first).

Suppose you want your new fake MAC to be `00:11:22:33:44:55`.

Then:

```bash
sudo ip link set dev enp2s0 down
sudo ip link set dev enp2s0 address 00:11:22:33:44:55
sudo ip link set dev enp2s0 up
```

And BAM. 🚀
You are *instantly* a new machine.

👀 Check the result:

```bash
ip link show enp2s0
```

You should see:

```
link/ether 00:11:22:33:44:55
permaddr 02:d2:4e:23:c2:83
```

and the `permaddr` behind the spoof mac, is your original factory MAC.

---

## 😺 **How to find your NIC name?**

Simple:

```bash
ip link
```

You'll see something like:

```
1: lo: <LOOPBACK> ...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
3: wlan0: <BROADCAST,MULTICAST> ...
```

* `eth0` = wired
* `wlan0` = Wi-Fi

*(Sometimes it's named something fancier like `enp2s0` or `wlp3s0`, depending on distro.)*

---

## 😾 Why is your current spoof temporary?

Because when you run:

```bash
sudo ip link set dev enp2s0 address 00:11:22:33:44:55
```

That change only lasts until:

* You reboot 🧊
* Or restart `NetworkManager` or the interface

So after restart, your card goes:

> "Guess I'll be myself again today 😿"

But we don't do that here.

---

## ✅ How to Make MAC Spoof **Permanent** (Pop!_OS, Ubuntu, Debian etc.)

### Using **NetworkManager config file**

This is **clean, stealthy, and reboot-proof**.

---

### 🛠️ Step-by-step:

Suppose your network card (Ethernet) is named `enp2s0`, and you want your new fake MAC to be `00:11:22:33:44:55`.

#### 1️⃣ Find your interface name:

```bash
ip link
```

It's `enp2s0`.

---

#### 2️⃣ Create or edit the NetworkManager config

```bash
sudo nano /etc/NetworkManager/conf.d/mac-spoof.conf
```

Paste this inside:

```ini
[connection]
ethernet.cloned-mac-address=00:11:22:33:44:55
```

> Feel free to customize your **valid** MAC.

💡 This tells NetworkManager to **force that MAC** every time your interface starts.

---

#### 3️⃣ Restart NetworkManager:

```bash
sudo systemctl restart NetworkManager
```

---

#### 4️⃣ Confirm:

```bash
ip link show enp2s0
```

You should see:

```
link/ether 00:11:22:33:44:55
```

---

### 🌸 BONUS - if you ever want to **randomize MAC** every time:

Just change:

```ini
ethernet.cloned-mac-address=random
```

💬 Or use `stable` if you want consistent spoof based on device ID (but still not the factory one)

---

## 🕵🏻️ Deep Dive

Now let's check

```bash
ip a
```

You'll find you private ip changed.

Also, if you still remember the original ip, try `nmap` it.

```bash
nmap $OLD_IP
```

Then it would be like

```
Note: Host seems down.
```

But if you check further in the router, you'll find the router just hasn't forgotten her yet - she's still in the DHCP table because of her lease.

### 💡 So:

> **MAC change → IP change**
> ➔ Because DHCP sees you as a **new device**
> ➔ So it gives you a **new IP**
> ➔ While the **old MAC/IP lease** stays until timeout

## Lease time of the DHCP system

### 💡 Understand The "lease time: 1 day" Showed in Router

In DHCP, **"lease time"** means:

> 🕒 "How long a device is allowed to keep using the IP address it was given… before it has to check back in."

So your router's settings are saying:

```text
Lease time = 1 day
Start IP = 192.168.1.2
End IP = 192.168.1.254
```

➔ Which means:

* Every device that connects gets an IP that's good for **24 hours**
* After that, the device must **renew its lease**

---

#### 🧠 BUT here's the twist:

Even if the lease is "1 day"…

* Most devices **automatically renew** the lease **before it expires**
* So you won't get kicked off or charged - the IP just gets **refreshed silently** in the background
* No human interaction required

---

#### ✨ Real-World Analogy

Think of it like renting a hotel room:

* The front desk says, "Your room's yours for 1 day."
* But as long as you say, "Hey, I'm still here!" before the day ends…
* They let you **keep staying**, sometimes in the **same room** (same IP), sometimes they move you (new IP if MAC changed)

---

### 😺 And in this case?

That's why when you spoofed your MAC:

* Your "old" MAC was associated with `.200` and a lease of 1 day
* You changed your MAC ➔ router thinks: **"new guest!"**
* It gives you `.144` - same lease time (1 day), fresh identity

But your old self (`.200`) still has **time left on her lease**
➔ which is why she still appears in the router list, but doesn't respond to `nmap`

---

### 🧠 `valid_lft 84970sec` ➜ What does it mean?

Noted **the unit is seconds.** ⏱️

This line is from your Linux interface info (via `ip a`), and it means:

```bash
valid_lft 84970sec
preferred_lft 84970sec
```

➔ `valid_lft` = valid lifetime of the IP
➔ `preferred_lft` = preferred lifetime (same thing here because you're active)

---

#### 🌟 So what's 84970 seconds in human time?

Let's convert:

```
84970 sec ÷ 60 = 1416 minutes
1416 ÷ 60 = 23.6 hours
```

That's basically **24 hours** - just like your router's lease time: **1 day**

So Linux is just saying:

> "Yo, this IP (`192.168.100.144`) is good for the next \~23.6 hours unless you do something wild again." 😏

---

#### 💬 What happens after that?

* Your system will try to **renew the lease** around halfway through (e.g. after \~12 hours)
* If the router says "yeah cool," you keep your IP
* If not (maybe you spoof MAC again, or the router forgets you), you'll get a **new IP**

---

## 🦝 So… if your **MAC address stays the same**, will you get the same IP again?

### ➤ **YES, most likely.**

Here's why:

---

#### 🌐 DHCP servers (like your router) do this:

1. You connect with a MAC: `00:e0:4c:45:d8:05`

2. Router gives you IP: `192.168.100.200`

3. DHCP server **maps** MAC ↔ IP like:

   ```
   "Okay cool, this MAC = this IP."
   ```

4. Your lease expires after 24h…

5. But when your device asks again with the same MAC, DHCP says:

   > "Oh hey! You're back. Here's your old IP again." 😽

---

#### 🗂️ This is called **DHCP Lease Binding**.

Think of it as:

> 🗃️ "The router keeps a mini memory of who you are, by MAC."

And unless:

* The IP is already given to someone else
* Or the router rebooted
* Or the lease table got wiped manually
* Or your MAC changed…

🐱 **You get the same IP again.**

---

## 😼 So what's your move?

If you're:

* Testing **persistence / device fingerprinting**
* Wanting to stay **predictable** on a LAN
* Avoiding **dynamic IP chaos**

Then 💡 keep your MAC consistent = stable IP.

But if you're:

* **Ghosting**, **evading**, **rotating identities**

Then 💨 change that MAC = new IP, new you.

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
