---
layout: post
title: "Bend the LAN to Your Will - Building Your Own Subnet from Scratch"
description: "From `255.255.255.0` to full-blown subnet royalty - this blog walks you through setting up your own `/29` subnet, manually assigning IPs, running a hotspot router from your laptop, and hosting a stealthy local network."
date: 2025-05-21 00:44:00 +0800
categories: [🤖 tech, 🔒 Web Security]
tags: [🐧 Linux, 💻 Networking, 🐾 Hacker Basics, 📡 DHCP, 🛡️ IP Refresh, 🔄 NAT Routing, 🧠 Subnet, 🔥 IP Masquerading, 🛰️ Local Server, 👾 nmap, 🧠 OSI, 🌐 LAN]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/subnet.png
---

Try thinking **like the router** now. Let's dig in deep...

So normally people set the subnet mask to `255.255.255.0 (/24)`.

But does anyone ever use something *different*?

Like, would that *limit how many devices can connect* to the router - and also limit the *available IPs*?

---

## 😼 Let's Talk About Subnet Mask!

### 🧠 So… why is `255.255.255.0 (/24)` so common?

It's because:

* It gives **256 total IPs** in that subnet
* Of those, **254 usable IPs** for devices
* Easy to manage, fits most home/small office setups

This is the **default** on most routers:

```
192.168.0.0/24  ➜ usable IPs: 192.168.0.1 → 192.168.0.254
```

That's why most networks feel like:

> "You and like 253 of your invisible roommates." 🫣

---

🌟 But people *can* change it.

And **some do**, especially network admins, hacker setups, or big corps.

---

#### 🧠 Let's break down the options:

| Subnet Mask       | CIDR  | Usable IPs | Used When...                                            |
| :---------------- | :---- | :--------: | :------------------------------------------------------ |
| `255.255.255.0`   | `/24` |     254    | 🏠 Home routers / small networks                        |
| `255.255.255.128` | `/25` |     126    | Smaller networks, segmented security zones              |
| `255.255.255.192` | `/26` |     62     | IoT/guest isolation zones                               |
| `255.255.255.248` | `/29` |      6     | 🧪 Pentest labs, VM networking                          |
| `255.255.254.0`   | `/23` |     510    | 🔥 Slightly bigger LANs (like dorms, offices)           |
| `255.255.0.0`     | `/16` |   65,534   | 🚨 Corporates / crazy setups (don't do this at home 😅) |

---

### 🌸 So why would someone **set it smaller** (like `/30`)?

To limit:

* The number of devices allowed to connect
* The **attack surface** (fewer IPs = fewer targets)
* Or for **precise segmentation** in a hacker lab or enterprise

🔐 **Smaller subnets = tighter control.**

e.g.:

* You create a `/29` subnet = 6 usable IPs
* You assign:

  * 1 for router
  * 2-3 for your servers
  * The rest? *Either unused, or honeypots*

---

### 🌀 A Small Preview

If you spoofed your IP like:

```bash
sudo ip addr add 192.168.66.66/29 dev enp3s0
```

You just joined a network where the only usable IPs are:

```
192.168.66.1 to 192.168.66.6
```

💬 Everyone else?
Blocked. Denied. No party for them.

---

#### `sudo ip addr add 192.168.66.66/29 dev enp3s0`

Let's break this command open like a juicy cyber snack 😈


```bash
sudo ip addr add 192.168.66.66/29 dev enp3s0
```

🌟 Translation in plain English:

> **"Hey Linux, please give my network card (enp3s0) a new IP address - 192.168.66.66 - and tell it we're on a tiny subnet (/29)."**

🔹 `ip addr add`

➜ Tells Linux:

> "Add this IP address to one of my network interfaces."

✨ You can assign multiple IPs to the same card - this is called **IP aliasing**.

---

🔹 `192.168.66.66/29`

> ❗ This is the **IP address you're giving yourself**, plus the **subnet size**.

* IP: `192.168.66.66` → this is your **identity** in the new subnet
* `/29` ➜ subnet mask = `255.255.255.248`

✨ A `/29` subnet = only **6 usable IPs** in total:

```
192.168.66.65 - 192.168.66.70
```

(Why not 8? Because 1 is network address, 1 is broadcast.)

You've basically said:

> "I'm going to exist inside this tight little hacker LAN with only a few trusted friends... or targets."

---

This created:

* A new *IP identity* on a new *subnet*
* Without touching your main one
* Now your machine can **talk to other machines in the 192.168.66.64/29 network**
* Even if it's virtual, isolated, or just for experiments

---

#### 🧷 Is it safe to assign yourself a second private IP?

✅ Yes, 100% safe - and super useful.

You're not *changing* your primary identity,
you're just **adding a second one**, like having a **dual passport**.

---

#### 🧾 Can I add more than one IP to enp3s0? What's the limit?

HELL. YES.

You can assign as many IPs to your interface as:

* Your **kernel memory** can handle
* And you don't go insane managing it 😹

Linux doesn't impose a strict limit - but you can do:

```bash
sudo ip addr add 192.168.66.67/29 dev enp3s0
sudo ip addr add 192.168.66.68/29 dev enp3s0
sudo ip addr add 10.0.0.42/24 dev enp3s0
```

They can all coexist.

✨ Use cases:

* **Impersonate multiple devices**
* **Emulate a network of victims**
* **Spin up multiple containers/services bound to different IPs**

⚠️ Just remember to:

* Avoid IP conflicts
* Use different subnets carefully (routing might get weird otherwise)

---

#### 💡 What can you do with anther private IP?

| Purpose                        | Example                                                |
| :----------------------------- | :----------------------------------------------------- |
| 🧪 Build a fake network        | Test servers, containers, sniffers in a private zone   |
| 🧑‍💻 Create internal services | Like a C2 server or honeypot accessible only by you    |
| 💥 Hide dual identities        | Respond to scans from different IPs, confuse attackers |
| 🎮 Simulate a LAN party        | Pretend you have 2+ hosts from one machine             |
| 📡 Set up firewall testing     | Block/allow per subnet - perfect for defense practice  |

You live in two subnets simultaneously.
You can scan yourself (like with `nmap 192.169.66.65/29`). Spy on yourself. Attack yourself. 👾

---

## ➕ Let's Do Some Easy Math!

### 🦝 How do we calculate usable IPs from a CIDR like `/29`?

CIDR (`/29`) defines **how many bits** of the 32-bit IP are used for **network addressing**.

---

#### 🧠 Formula:

```
Usable IPs = 2^(32 - CIDR) - 2
```

💡 Why minus 2?

* One is **network address** (first IP)
* One is **broadcast address** (last IP)

---

#### ✨ Examples:

| CIDR | Subnet Mask     | Total IPs | Usable IPs |
| :--: | --------------- | :-------: | :--------: |
|  /30 | 255.255.255.252 |     4     |      2     |
|  /29 | 255.255.255.248 |     8     |      6     |
|  /28 | 255.255.255.240 |     16    |     14     |
|  /24 | 255.255.255.0   |    256    |     254    |
|  /16 | 255.255.0.0     |   65,536  |   65,534   |

🧪 So for `/29`:

* IP range: `192.168.66.64 - 192.168.66.71`
* Network IP: `192.168.66.64` ❌
* Broadcast IP: `192.168.66.71` ❌
* ✅ Usable IPs:

  ```
  192.168.66.65
  192.168.66.66
  192.168.66.67
  192.168.66.68
  192.168.66.69
  192.168.66.70
  ```

---

#### 🧮 "What about CIDR `/32`?"

`/32` is a **valid CIDR**, and it's special:

* `255.255.255.255`
* Means: **"this exact single IP address"**
* 0 usable IPs? **Yes - because it's reserved for identity, routing, or loopback**

You'd use `/32` to:

* Refer to a **host** in a route:
  ➜ "This IP only routes to `192.168.1.33/32`"
* Lock firewall rules:
  ➜ "Only allow SSH from `10.0.0.7/32`"
* Or for **local loopback** stuff

So - `2^(32-32) - 2 = 0 - 2 = -2` is **theoretical**,
but **you're not meant to assign multiple devices to `/32`**. It's **for single-IP logic only.**

---

### 📊 Why is IP address 32-bit?

IPV4 = **32 bits** total = 4 bytes = 4 octets

Each IP address (like `192.168.1.1`) is really:

```text
11000000.10101000.00000001.00000001
```

Each octet = 8 bits → `8 × 4 = 32 bits` total.

🔐 That gives us:

```
2^32 = 4,294,967,296 total possible IPv4 addresses
```

---

## 🗾 How to Interrogate Your Subnet with `nmap`?

**You can use *any IP* in the subnet to `nmap` with.**

### Because `nmap` with `/29` means:

> "Scan all 8 IPs in the `192.168.66.64/29` range, no matter where I start from."

---

### ✅ The Real Magic

`nmap` interprets the **CIDR mask**, not just the starting IP.

So all of these are equivalent and will scan the exact same range:

```bash
nmap 192.168.66.64/29
nmap 192.168.66.65/29
nmap 192.168.66.66/29
nmap 192.168.66.67/29
nmap 192.168.66.70/29
```

All of them will scan:

```
192.168.66.64
192.168.66.65
192.168.66.66
192.168.66.67
192.168.66.68
192.168.66.69
192.168.66.70
192.168.66.71
```

🧠 Because `/29` says:

> Subnet mask = `255.255.255.248`
> Meaning: 8 IPs, starting from the nearest multiple of 8 *below or equal to your input IP*

So even if you enter `192.168.66.69/29`, it'll still calculate:

```
Base: 192.168.66.64  →  + 8 IPs
```

---

### ⚠️ But keep in mind

| IP              | Role                                           |
| --------------- | ---------------------------------------------- |
| `192.168.66.64` | Network address (unreachable)                  |
| `192.168.66.71` | Broadcast address (won't reply to pings/ports) |

So realistically, **only `.65` to `.70`** might respond.

---

### ✅ Best Practice Scan:

```bash
nmap -sn 192.168.66.64/29
```

* `-sn` → "Ping scan" (don't port scan, just host discovery)

Want to see open ports too?

```bash
nmap -p 22,80,443 192.168.66.64/29
```

> 💡 "Devices won't always reply to ping if ICMP is disabled - try `-Pn` if a host seems down but you suspect otherwise."

---

## 🧝🏻 A Touch of Magic: Hosting with `python3 -m http.server`

```bash
python3 -m http.server --bind <your_ip> 8080
```

Which means:

> "Run a simple HTTP server (a tiny website!) on **port 8080** and **bind it to this IP address only**."

Now, **any device in the same subnet** that can route to that IP\:PORT combo can access your ENTIRE working directory in a browser.

---

### 💡 Let's break it down:

| Thing                    | What it does                             |
| :----------------------- | :--------------------------------------- |
| `python3 -m http.server` | Starts a basic file server over **HTTP** |
| `--bind 192.168.X.X`     | Binds to a **specific IP address**       |
| `8080`                   | TCP port number (browser will look here) |

Suppose your PC's private ip is `192.168.1.73`, 

So when you visit:

```
http://192.168.1.73:8080/
```

your **Phone is sending a GET request** to:

* IP `192.168.1.73`
* Port `8080`
* Using protocol `HTTP`

Then your PC replies with:

> "Yo 👋 Here's the entire folder contents of wherever I ran that command." 

---

### 🕸️ Tech Layers Behind the Magic:

When you accessed your Python HTTP server from your Phone,
you activated **four layers of the OSI model** to make that one magical moment happen:

---

| OSI Layer             | What was happening?                                    |
| :-------------------- | :----------------------------------------------------- |
| **🌐 L7 - Application (HTTP)**  | HTTP server runs in Python → serving files             |
| **L6 - Presentation** | Browser formats your file list nicely (HTML rendering) |
| **L5 - Session** | 	TCP session established - a persistent, stateful connection |
| **🔢 L4 - Transport (TCP)**    | TCP handshake on port 8080 (`SYN, SYN-ACK, ACK`)       |
| **🌍 L3 - Network (IP)**      | IP routing between your Phone & PC                  |
| **L2 - Data Link**    | MAC address & ARP resolving                            |
| **🧭 L1 - Physical**     | Your Wi-Fi/Ethernet transmitting radio signals ⚡       |

So that moment?
It wasn't just "oh a browser opened a folder."

It was your phone and PC **cooperating across 5 network layers**,
dancing the protocol tango like pros. 👾

✨ This is classic **Layer 7-3 cooperation**. Your stack is alive and thriving.

---

### 🔓 Why this Feels Like a Magical Discovery

Because with this, you could:

* Ran a **public-facing server**
* Saw your own **IP as a host**
* Understood that any device *on the same subnet* can reach you via IP\:PORT
* Realized you're not just a *client* - you're now a **node**. A **server**. A **target**. A **controller**.

> 💡 Any device with a known IP and open port becomes instantly reachable by any peer on its subnet.

---

## 🧠 Build Your Own Custom `/29` Subnet and Link Up Your Phone, Laptop, and More

### Check for Wi-Fi interface

Most desktop PCs (especially gaming/workstation towers) **don't come with a Wi-Fi card** unless:

* You **explicitly installed** a PCIe Wi-Fi card
* Or you're using a **USB Wi-Fi dongle**

Install the tool if you haven't:

```bash
sudo apt install iw
```

So if you run:

```bash
iw list
```

…and it shows **nothing** or no `AP` mode?

Then yeah - **you have no Wi-Fi interface** at all
or the one you have **doesn't support Access Point mode.**

---

#### 🧪 Double-check what interfaces you have

```bash
ip link
```

If you only see:

```bash
lo
enp3s0
```

(no `wlan0`, `wlp*`, or `wlan1`)
then **you 100% don't have a Wi-Fi card plugged in**.

> So the question is whether to set up a Wi-Fi access point on your PC, or just build it using a laptop that already has Wi-Fi and can run things like hostapd, subnet empires, and more.

😾 Here I'll go with laptop and iPhone.

---

### 🤨 Who Should Be the Router Now?

When you tether your laptop to an iPhone, you get dropped into a separate subnet (172.20.10.x) that's NATed and isolated - perfect for sneaky or secure file transfers. That subnet **isn't visible** to other machines on your corporate LAN.

It's basically like your **iPhone becomes the router**, and your laptop is sitting quietly inside that subnet. Way safer when you're at school, work, or anywhere you want some real **privacy**.

This is **exactly what advanced security-minded folks** do:

*  Use a tethered iPhone

*  Transfer tools/files to/from phone

*  Stay invisible to IT monitoring

But here's the twist:

What we actually want is to build a *custom* subnet - something like 192.168.66.65/29.  
The problem? You can't easily reassign the iPhone's hotspot IP range.  
So yeah… we'll need to mess with the laptop instead, aka **let your laptop be the router**.

---

### 🚥 Activate the Hotspot on Laptop

Before we go any further, to avoid IP conflicts, make sure you haven't already assigned the IP we plan to give the laptop to any of your Ethernet interfaces.

If you have, clear it out with:

```bash
sudo ip addr flush dev eth0
sudo dhclient eth0
```

*(Replace `eth0` with your actual interface name.)*

Then start the hotspot you've created it from the GUI

```bash
nmcli connection up <connection-name>
```

---

#### Hotspot Setup

| Setting           | Status                      |
| ----------------- | --------------------------- |
| Mode              | Access Point                |
| WPA2 password set | via Wi-Fi Security          |

1. Edit the hotspot profile:

   * IPv4 Method: **Manual**
   * Assign your laptop's static IP, e.g.

     | Address         | Netmask           | Gateway         |
     | --------------- | ----------------- | --------------- |
     | `192.168.66.66` | `255.255.255.248` | *(leave blank)* |

2. Save the profile. If it complains like:

> `method 'manual' requires at least an address or a route`

✅ Just make sure to fill *all 3 fields* with something like:

```
192.168.66.66    255.255.255.248    192.168.66.66
```

3. ✅ Check "IPv4 is required for this connection"

---

### run a real dnsmasq

✅ Step 1: Install dnsmasq

```bash
sudo apt update
sudo apt install dnsmasq
```

This will:

* Install `dnsmasq`
* Create `/etc/dnsmasq.d/` directory
* Register the `dnsmasq.service` with `systemd`

✅ Step 2: Enable & start it

After installing:

```bash
sudo systemctl enable dnsmasq
sudo systemctl start dnsmasq
```

If you already wrote your config like:

```ini
# /etc/dnsmasq.d/my-subnet.conf
interface=wlan0
dhcp-range=192.168.66.66,192.168.66.70,255.255.255.248,12h
dhcp-option=3,192.168.66.66   # Gateway
dhcp-option=6,8.8.8.8         # DNS
```

Then just:

```bash
sudo systemctl restart dnsmasq
```

---

### NAT forwarding

Right now:

* You've got a working DHCP subnet (`192.168.66.64/29`)
* iPhone gets IP and connects fine
* But no internet access, because **there's no NAT (masquerading) from wlan0 → eth0**

There it is - classic **NAT forwarding problem**.

---

#### 🛠️ FIX: Enable NAT with iptables

```bash
sudo sysctl -w net.ipv4.ip_forward=1

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Explanation:

| Command        | Purpose                                                       |
| -------------- | ------------------------------------------------------------- |
| `ip_forward=1` | Allows laptop to route packets between networks                 |
| `MASQUERADE`   | Translates packets from iPhone to look like they're from laptop |

If it works and you want to make it **permanent** next boot:

---

#### Enable IPv4 forwarding:

```bash
sudo nano /etc/sysctl.conf
```

Uncomment or add:

```ini
net.ipv4.ip_forward=1
```

---

#### Save NAT rule permanently:

Install iptables-persistent:

```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

Now try to test if iPhone get the Internet. If something went wrong here, try restarting NetworkManager.

```bash
sudo systemctl restart NetworkManager
```

Wait a few seconds. Then retry:

```bash
nmcli connection up <connection-name>
```

---

##### If it doesn't save the rule when reboot:

✅ 1. make ip forwarding permanent

Edit this file:

```bash
sudo nano /etc/sysctl.conf
```

Make sure you have:

```ini
net.ipv4.ip_forward=1
```

Then ALSO create a config in `/etc/sysctl.d/99-hotspot.conf` to override anything ignored:

```bash
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-hotspot.conf
```

Apply instantly:

```bash
sudo sysctl --system
```

✅ Verify:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

→ Should say `1`

This double method ensures it's applied at boot - even if `NetworkManager` or some weird systemd thing resets `/proc`.

✅ 2. ensure NAT rules load on boot

You've installed `iptables-persistent` - great!

Now run this to check your saved NAT rule:

```bash
sudo iptables -t nat -L POSTROUTING
```

You should see:

```
MASQUERADE  all  --  anywhere  anywhere
```

If it's there - now force reload on boot:

```bash
sudo systemctl enable netfilter-persistent
```

This ensures the NAT rule gets re-applied *after all interfaces are up* at boot time.

---

### 📱 Manually configure static IPs for other devices

Wanna customize your private IP?

On iPhone:

* IP: `192.168.66.67`
* Mask: `255.255.255.248`
* Gateway: `192.168.66.66`
* DNS: `8.8.8.8` *(do not make it auto! must force a public resolver)*

This(especially the `DNS`) tricks iOS into "trusting" the static connection.

---

Phew, so finally the whole thing would be like:

* Laptop: `192.168.66.66`
* Phone: `192.168.66.67`
* Tablet: `192.168.66.68`
* Subnet mask: `255.255.255.248`
* Gateway: `192.168.66.66` (your host laptop)

---

## 🧠 What We Have Done in This Blog

We didn't just talk theory - we **built** a functioning, custom subnet from scratch:

* 🛠️ Created a `/29` subnet with **tight IP allocation**
* 🔌 Configured **manual IP addressing** for fine-grain control
* 🧑‍🍳 Used `dnsmasq` to cook up our own DHCP service
* 🧱 Set up **NAT and IP forwarding** to let subnet clients access the Internet
* 🔍 Scanned & verified subnet activity with `nmap`
* 📡 Hosted a mini HTTP file server to test access
* 🧠 Understood the **OSI stack magic** happening underneath every request

You went from **"what's a subnet?"** to **"I built a subnet empire where I am the god of DHCP and gateway."**

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
