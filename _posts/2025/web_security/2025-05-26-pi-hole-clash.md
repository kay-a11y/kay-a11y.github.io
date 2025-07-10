---
layout: post
title: "Building a Pi-Hole DNS Firewall with Clash VPN Routing - Wireless, Leak-Free, No-Cable Setup"
description: 
date: 2025-05-27 18:40:00 +0800
categories: [🤖 tech, 🔒 Web Security]
tags: [🐧 Linux, 💻 Networking, 🍓 Raspberry Pi, 🚫 Ads Blocking, 🔄 NAT Routing, 📡 DHCP, 🛡️ DNS, 😼 Clash, 🌍 LAN Proxy, 🌀 VPN, 🧠 OSI, 🌐 LAN, 😨 IPv6]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/pi_hole.png
---

> **⚠️ WARNING:** If you're using a non-jailbroken iPhone, you need to use `Shadowrocket` to fit this Pi + Clash setup.  
> 📱 **Make sure your phone or router allows IPv6 to be disabled** *before* you commit to this setup.  
> **🪛 <a href="https://kay-a11y.github.io/posts/troubleshoot-clash-port/" target="_blank" rel="noopener noreferrer">check this post</a> for more troubleshooting.**

---

So here's the deal.  
I'm still pretty fresh to Raspberry Pi and all that terminal kung fu, but I made the damn thing **my personal digital sniper**-silently blocking ads, hijacking DNS, and routing all my traffic through encrypted tunnels like a stealthy lil fox in black hoodie mode.

> No more phucking ISP tracking.  
> No more random DNS leaks.  
> No more lame wired setups or HDMI tethering like it's 2009.

I'm not even gonna dignify that rotten excuse of a firewall with a name. Starts with G, ends in get the PHUCK outta my packets. 🖕  
We're playin' nice here - for the blog. But you know. You know.

---

This blog is for my fellow rebels who wanna:

* 📖 **Actually understand** how DNS works and why it's not just some invisible ghost
* 🛠️ Make **Pi-hole** and **Clash.Meta CLI** run side-by-side like baddies with knives
* 🚫 Keep ad-blocking strong on **DIRECT traffic**, even when things get tangled with proxies
* 🔌 SSH into the Pi **wirelessly**, like a damn boss-no HDMI, no cables, no mess
* 🧠 Build a setup that **routes traffic smart**, not blindly through your PHREAKIN' ISP
* 🔒 **Avoid DNS leaks** like the plague and keep your data where it belongs - **with you**

Trust me - once you taste this kind of setup freedom, you'll never wanna go back to vanilla internet.

---

## 🚠 Enable SSH

This would suppose you already finished installation of your Pi OS.  
First things first, before we start, make sure you enable the SSH service on your pi.

### 1. **Make sure SSH is installed and running on the Pi**

Now only this would need a HDMI to check, so boot Pi into GUI, let's do a double check:

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

> You wanna see: `active (running)` in green.  
> If it says *not found*, just do:

```bash
sudo apt update
sudo apt install openssh-server
```

---

### 2. **Get your Pi's IP address**

```bash
ip a
```

You'll get like `<PI_IP>`.

But to double-check:

```bash
hostname -I
```

---

### 3. **SSH from your Main PC**

Now you could get HDMI outta of your so darn sexy monitor. From your PC terminal:

```bash
ssh <USERNAME>@<PI_IP>
```

First time? Say yes to the fingerprint.  
Then you're in - your Pi, remotely seduced.

---

### 4. **Configure static IP on Pi**

If you don't wanna touch the router:

Edit your Pi's DHCP config:

```bash
sudo nano /etc/dhcpcd.conf
```

Scroll down and add:

```ini
interface wlan0
static ip_address=<PI_IP>/24
static routers=<Router_IP>
static domain_name_servers=1.1.1.1 8.8.8.8
```

Save it (Ctrl+O, Enter, Ctrl+X), then:

```bash
sudo reboot
```

---

## 🍍 install Pi-hole

### 🛠️ Step-by-step: Install Pi-hole

**Run this one-liner install command (curl version):**

```bash
curl -sSL https://install.pi-hole.net | bash
```

This will launch a guided terminal installer that will ask you:

---

### 💡 What you'll be asked (and what to pick)

1. **Static IP**  
   ✔️ click `<Continue>`

2. **Choose an interface**  
   🛜 Hit `space` to select `<whlan0>` whatever your wireless interface is - since we're going fully wireless with the Pi.  
   ❌ Make sure `eth0` is unselected (just one should be checked)

3. **DNS provider**  
   🕘 Start with Quad9 (filtered, DNSSEC) since we don't have a VPN set up yet.

   ☁️ Once the VPN's ready, switch to Cloudflare (DoH or DoT) for better privacy.

4. **Blocklists**  
   ✔️ Use the default ones - you can add more later.

5. **Web admin interface?**  
   ✔️ YES, you want this

6. **Web server (lighttpd)?**  
   ✔️ YES - unless you already plan to use Nginx

7. **Log queries?**  
   ✔️ YES - it's fun seeing what gets blocked

8. **Select a privacy mode for FTL**  
    0️⃣ Stick with `0 - Show everything` - Logs ALL domain requests + which client made them

---

### ✅ When it finishes

It'll give you a cute little message like:

> Web Interface: [http://<PI_IP>/admin](http://<PI_IP>/admin)
> Password: [your randomly generated web password]

Save that password or run:

```bash
pihole -a -p
```

to change it.

---

### 🔥 Now access it

Open a browser and go to:

```
http://<PI_IP>/admin
```

Login, poke around!

#### 😽 Recommend Blocklist

In `Lists` ➡️ `Address:` ➡️ `Add blocklist`

You can add these URL **one by one**. ⬇️

```txt
https://adaway.org/hosts.txt
https://big.oisd.nl/
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
https://raw.githubusercontent.com/PolishFiltersTeam/KADhosts/master/KADhosts.txt
https://urlhaus.abuse.ch/downloads/hostfile/
https://pgl.yoyo.org/adservers/serverlist.php?hostformat=hosts&showintro=1&mimetype=plaintext
https://raw.githubusercontent.com/Isaaker/Spotify-AdsList/main/Lists/pi-hole.txt
```

then

```bash
sudo pihole -g
```

or `Tools` ➡️ `Update Gravity` ➡️ `Update`

---

## 📱 Set your Phone/router's dns

### ✅ Set DNS manually in phone Wi-Fi settings

Go to your phone's **Wi-Fi > Network details > IP settings** → switch to **Static** or **Manual** and set:

* **DNS1**: `<PI_IP>` (your Pi-hole)
* **DNS2**: `<PI_IP>`

🧠 Pro Tip: Don't forget to **forget the network first**, then reconnect with new DNS.

---

## 💡 run a VPN on your Pi while still using Pi-hole as DNS

### 💬 TL;DR first

> You can run a **VPN (like Clash / V2RayN)** on your Pi while still using **Pi-hole as DNS** for all your devices, so:

* You block ads + trackers 🎯
* You **don't need to run VPN on every device** 🧘‍♀️

---

### ✅ Part 1: Confirmed - Pi-hole works perfectly **without VPN**

Try to connect some site already blocked by pi-hole via your phone/pc. If that works, that's great.

Which means your DNS chain looks like this:

```
[Device] --> [Pi-hole @ <PI_IP>] --> [Quad9 or other upstream]  
```

---

### ✅ Part 2: Why VPN breaks Pi-hole blocking

When you **run VPN** on your device, one of two things happens:

| Situation                                           | Result                                      |
| --------------------------------------------------- | ------------------------------------------- |
| VPN forces DNS over its own server                  | ❌ Pi-hole is bypassed entirely              |
| VPN sends all traffic (incl DNS) through its tunnel | 😢 Local ISP is bypassed, but ad blocking is gone |

So you're either *free* but *unprotected*, or *blocked* but *adless*.
That sucks. We want **BOTH**.

---

### 🔥 Part 3: THE MASTERPLAN 💻🌍

What you really want is this:

> 💡 **Use the Pi as a centralized VPN router** that:

* Runs **Clash**
* Routes all *international* traffic through the VPN
* Keeps **DNS local** so Pi-hole still blocks stuff
* Protects *all* devices in your network - **without installing VPN apps on them**

---

#### 🍓 Dream Setup Diagram

```
[All Devices]
      ↓ (DNS: <PI_IP>)
    [Router]
      ↓
    [Raspberry Pi]
    ├─ Pi-hole → handles DNS (blocks ads)
    └─ Clash → handles traffic routing
         ↳ sites in rule → VPN
         ↳ Local stuff → direct
```

---

#### 🛠 How to Set It Up

#### ✅ Install Clash on the Pi

Go to `https://github.com/MetaCubeX/mihomo/releases` download a latest release

e.g. :

```bash
# Download the binary, and don't forget to check out the latest release yourself!
wget https://github.com/MetaCubeX/mihomo/releases/download/Prerelease-Alpha/mihomo-linux-arm64-alpha-34de62d.gz

# Unpack it
gunzip mihomo-linux-arm64-alpha-34de62d.gz

# Rename it for convenience
mv mihomo-linux-arm64-alpha-34de62d clash

# Make it executable
chmod +x clash

# Move it into place
sudo mv clash /usr/local/bin
```

Now try:

```bash
clash -v
```

You should see something like:

```scss
Clash.Meta (mihomo) vX.XX...
```

#### 📁 Add your node/subscription file

Drop in your config here:

```bash
sudo nano /root/.config/mihomo/config.yaml
```

Paste in your config or YAML from your GUI-exported setup.  

---

Or copy the config into your Pi from your PC:

On PC:

```bash
scp <yaml_path> raspberrypi@<PI_IP>:~/config-temp.yaml

```

SHH into Pi, then move with sudo:

```bash
sudo mv ~/config-temp.yaml /root/.config/mihomo/config.yaml
```

---

#### change dns config

```yaml
mixed-port: 7890
redir-port: 7892
tproxy-port: 7893
allow-lan: true
bind-address: '*'
mode: rule
log-level: info

external-controller: '0.0.0.0:9090'
secret: "<your_password>"

dns:
  enable: true
  ipv6: false
  listen: ':1053'
  default-nameserver:
    - <PI_IP>
  enhanced-mode: fake-ip
  nameserver:
    - <PI_IP>
  proxy-server-nameserver:
    - <PI_IP>
  fallback:
    - <PI_IP>
  fallback-filter:
    geoip: true
    geoip-code: "<your_country_code>"
    geosite:
      - "<your_region_blocklist_or_category>"
```

> * Applying `<PI_IP>` in `proxy-server-nameserver` and `fallback` only work IF Pi-hole resolves external domains reliably.  
> * Use these ONLY if your Pi-hole's upstream DNS isn't trapped behind a firewall or failing requests.

> Feel free to change `<your_password>`.

### Run clash and select your node

Now ran

```bash
clash
```

---

#### ✅ **Pick your node**

1. **List groups**:

   ```bash
   curl -s -H "Authorization: Bearer <your_password>" http://127.0.0.1:9090/proxies | jq '.proxies | keys'
   ```

2. **Check what's inside `<Node Provider>`**:

   ```bash
   curl -s -H "Authorization: Bearer <your_password>" http://127.0.0.1:9090/proxies/<node_provider> | jq '.all'
   ```

3. **Switch to whatever node you want**, like:

   ```bash
   curl -X PUT -H "Authorization: Bearer <your_password>" -H "Content-Type: application/json" \
    -d '{"name": "<node_name>"}' \
    http://127.0.0.1:9090/proxies/<node_provider>
   ```

   > Feel free to change `<your_password>`, `<node_provider>` and `<node_name>` to your own values.

---

#### ✅ **Verify it worked**

```bash
curl --proxy http://127.0.0.1:7890 https://api.myip.com
```

If it says your proxy IP - then yes, you're surfing the free web.

---

### ✅ **Make Clash/Mihomo run in the background (so terminal can close)** - Use `systemd` for full background service

Let's make Mihomo auto-start on boot like a real daemon

1. Create a systemd service:

   ```bash
   sudo nano /etc/systemd/system/mihomo.service
   ```

   Paste:

   ```ini
   [Unit]
   Description=Mihomo Service
   After=network.target

   [Service]
   ExecStart=/usr/local/bin/clash -f /root/.config/mihomo/config.yaml
   Restart=on-failure
   User=root

   [Install]
   WantedBy=multi-user.target
   ```

2. Enable & start it:

   ```bash
   sudo systemctl daemon-reexec
   sudo systemctl enable mihomo
   sudo systemctl start mihomo
   ```

3. Check status:

   ```bash
   sudo systemctl status mihomo
   ```

### 🔥 IP forwarding & iptables Setup

#### Step 1: Enable IP forwarding on your Pi

  ```bash
  echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
  sudo sysctl -w net.ipv4.ip_forward=1
  ```

  Make it permanent by editing:

  ```bash
  sudo nano /etc/sysctl.conf
  ```

  Uncomment or add:

  ```
  net.ipv4.ip_forward = 1
  ```

---

### Step 2: Setup iptables rules

You're gonna redirect traffic from other devices to Clash.

**Example iptables setup:**

```bash
sudo iptables -t nat -N CLASH
sudo iptables -t nat -A CLASH -p tcp -j REDIRECT --to-ports 7892
sudo iptables -t nat -A PREROUTING -s 192.168.1.0/24 -p tcp -j CLASH
```

> Replace `192.168.1.0/24` with your LAN subnet

💡 This redirects **all TCP traffic** from LAN devices through Clash's redir port.

---

### 😼 Final Step: Set your Raspberry Pi as **default gateway** for your phone

#### 🧪 Manual Gateway Setup on your Phone

On your phone's Wi-Fi settings:

1. Choose your current network
2. Edit **IP Settings** → set to **Static**
3. Then set:

   | Field      | Value                                                   |
   | ---------- | ------------------------------------------------------- |
   | IP Address | Same as your phone already has (e.g. `192.168.1.200`) |
   | Gateway    | `<PI_IP>` ← your Pi                             |
   | DNS        | `<PI_IP>` ← your Pi-hole                        |

4. Save and reconnect

---

#### 🔁 Now test

```bash
curl https://api.myip.com
```

From your phone (or go to [ipinfo.io](https://ipinfo.io)), and…

> 🥁 You should see your **proxy IP**, NOT your local ISP.

---

#### ☑️ Switch to Cloudflare

Now the VPN's ready, switch your Pi-hole DNS setting to Cloudflare (DNSSEC) for better privacy.

In `Settings` ➡️ `DNS` ➡️ Check these two boxes

![Cloudflare_DNSSEC](/assets/img/posts/Cloudflare_DNSSEC.png)

---

#### 🌷 What just happened?

Before:

```
Phone → Router → internet
```

After:

```
Phone → Pi (iptables trap!) → Mihomo → Proxy node → Free internet
```

---

## 🔍 What DNS Really Does

DNS = Domain Name System
It's like the **phonebook of the internet**-turning names like `spotify.com` into IP addresses like `35.186.224.25` so your device knows where to connect.

---

### 💡 So When You Use Pi-hole as DNS

* **Pi-hole becomes the phonebook** 📖
* Anyone using your Pi-hole to resolve DNS (like your Phone) is asking **your Pi-hole**:

  > "Yo, what's the IP of `doubleclick.net`?"

And your Pi-hole can:

* 🧱 Block domains you blacklist
* 🕵️ Log every domain someone asks
* 🔄 Forward to upstream DNS if needed

---

### 🔓 What's a DNS Leak Then?

> You can check [BrowserLeaks](https://browserleaks.com/dns) or [DnsLeakTest](https://www.dnsleaktest.com/) to see if you're experiencing a DNS leak.

A **DNS leak** happens when:

* Your device is **supposed** to use your DNS server (like your Pi-hole)
* But instead, it **sends DNS queries elsewhere**, usually:

  * Your ISP
  * Google (8.8.8.8 or `74.x.x.x`)
  * Some weird encrypted DNS behind your back (DoH/DoT)

This **leaks your browsing activity**, even if you're on a VPN or using a filter like Pi-hole.

---

## 🧱 OSI Model Breakdown

### 📡 **Layer 3 - Network Layer**

* This is where **IP routing** lives.
* When you **change the gateway** to your Pi, you're **redirecting packets** to go through your VPN tunnel on the Pi.
* That means: You're redirecting traffic away from your ISP before it ever reaches them.

> 💬 **Result**: Your traffic doesn't even touch the phucking local ISP. Pi hijacks it, puts it into the VPN, and tunnels it safely like a boss.

---

### ✨ **Layer 7 - Application Layer**

* DNS lives up here (yes, even though it *feels* low-level).
* Your Pi-hole acts here: **intercepts DNS queries** and either resolves or blocks them.
* Ads blocked? Domains rewritten? That's Layer 7 magic.

> Combined with Layer 7 control (DNS, ad-block, tracking protection), it's like you built your own damn **freedom stack** 🗽

---

### 🛠️ So What You've Done Is

| Layer | Action                                                             |
| ----- | ------------------------------------------------------------------ |
| 3     | **Redirected TCP/IP traffic** to your VPN tunnel on Pi             |
| 7     | **Intercepted DNS queries** and blocked/sanitized them via Pi-hole |

---

## Troubleshoot

If you've ever used `sudo pkill clash` `sudo systemctl stop mihomo`, then you probably got booted from SSH like *a phreakin' hot potato*. 🥔 (Don't worry, we've all been that potato. )

like:

```bash
ssh raspberrypi@<PI_IP>
ssh: connect to host <PI_IP> port 22: Connection refused
```

---

### 🛠️ How to fix `Connection refused`

#### Option 1: Unplug and replug Pi, if you already set Mihomo to auto-start as shown above.

#### Option 2: HDMI Cable

✅ Step 1: **Physically access the Pi**

Plug it into a monitor + keyboard, or connect via serial console if you have UART cables.

Log in locally.

---

✅ Step 2: **Re-enable networking**

Run this to see if networking still exists:

```bash
ip a
```

If `wlan0` or `eth0` doesn't have an IP like `192.168.1.184`, you can restart DHCP:

```bash
sudo dhclient wlan0
# Or for ethernet:
sudo dhclient eth0
```

If you have a static IP configured in `/etc/dhcpcd.conf`, check that it's not misconfigured or being ignored.

---

### ✅ Step 3: Restart SSH

```bash
sudo systemctl restart ssh
```

Then try to connect again from your PC:

```bash
ssh raspberrypi@<PI_IP>
```

---

### ✅ Step 4: Restart Clash safely

```bash
sudo systemctl start mihomo
```

And test with:

```bash
curl --proxy http://127.0.0.1:7890 https://api.myip.com
```

---

### ✅ Step 5: **reapply your iptables rules** after killing Clash

When you run:

```bash
sudo pkill clash
```

your iptables rules (especially custom NAT chains) may **not persistent**:

So reapply this:

```bash
# 💣 Flush existing NAT rules to avoid duplicates
sudo iptables -t nat -F
sudo iptables -t nat -X

# 🔁 Recreate CLASH chain
sudo iptables -t nat -N CLASH
sudo iptables -t nat -A CLASH -p tcp -j REDIRECT --to-ports 7892
sudo iptables -t nat -A PREROUTING -s 192.168.100.0/24 -p tcp -j CLASH
```

Then traffic from **all LAN devices** will be transparently routed through your **Clash TProxy setup** again.

---

### ✅ Step 5:  Make It Stick (Persistence Setup)

1. Install persistent rules

```bash
sudo apt update
sudo apt install iptables-persistent
```

Say **yes** when it asks to save your current rules.

---

### 🔥 Step 2: Save your working rules to disk

```bash
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

Wanna check them again?

```bash
sudo cat /etc/iptables/rules.v4
```

Now they'll load on **every reboot**, even if you're sleepy and forget 😪

> **👻 If DNS breaks, ads return, or VPN feels flaky:**
>
> * Restart `mihomo` & Pi-hole
> * Check `/etc/resolv.conf`
> * Clear device DNS cache
> * Or just... turn it off and back on
> * Maybe you'd like to check this [Troubleshooting](https://kay-a11y.github.io/posts/troubleshoot-clash-port/) out? 😿

<div style="display: flex; justify-content: center; align-items: center; margin: 1em 0;">
  <div style="position: relative; display: inline-block; width: 150px; height: auto;">
    <img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png"
         alt="Support me on Ko-fi"
         width="150"
         loading="lazy"
         style="display: block; width: 80%; height: auto;">
    <div onclick="window.open('https://ko-fi.com/kikisec', '_blank')"
         style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: transparent; cursor: pointer;">
    </div>
  </div>
</div>
