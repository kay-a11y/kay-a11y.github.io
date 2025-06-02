---
layout: post
title: "Proxy or Die Trying: Troubleshooting of Ports, Git, and Clash on Linux"
description: "A hands-on walkthrough of debugging Clash's HTTP & SOCKS proxy ports on Linux, configuring git/pip access, and switching ports with custom shell aliases."
date: 2025-05-28 22:38:00 +0800
categories: [🤖 tech, 🔒 Web Security]
tags: [🐧 Linux, 💻 Networking, 🍓 Raspberry Pi, 🛡️ DNS, 😼 Clash, 🌀 VPN, 🚪 Port, 🧙 Git, 🔁 HTTP/HTTPS Proxy, 🕳️ SOCKS5, 🐞 Linux troubleshooting]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/clash_troubleshooting.png
---

So let's say you've successfully set up Pi-hole for all your devices-your Phone, your PC, everything - maybe using this blog 👉🏻 [Building a Pi-Hole DNS Firewall with Clash VPN Routing - Wireless, Leak-Free, No-Cable Setup](https://kay-a11y.github.io/posts/pi-hole-clash/).  

But then... the network gets way more complex than I ever expected.  
Back when things were simpler, I could handle everything just fine.  
But BOOM - packets started getting messed up, ports got phucked. ⛓️‍💥  

Don't panic.  
I finally managed to troubleshoot everything after several hours (and yeah, I'm still kinda confused about what's actually happening under the hood).  
Hopefully, anyone reading this can solve similar issues too - *and faster than I did!!* 🐣

---

## ☠️ The Phreaking Trouble

* Clash Verge *failed to update the subscription file* when **TUN** was enabled - like it couldn't reach the internet at all. All nodes showed *timeout*.
* Switched to **system proxy** - file update worked, but nodes still timed out. Or sometimes connected, but IP wasn't the node's.
* Either way, when I hit `Test All` in Clash Verge, it didn't even show the loading circle - just failed.
* After setting **SOCKS5 Proxy in Firefox**, all apps and non-proxy (DIRECT) sites became inaccessible, even router IPs like `192.168.1.1`.
* When I turned off Firefox SOCKS5 Proxy, Firefox worked normally (or couldn't access foreign sites if you're in a place with **extremely phreaking insane** internet restrictions).

---

## 🦊 Reconnect to Your VPN on PC (The Fast & Dirty Fix) - Set a SOCKS5 Proxy in Firefox

Let's say Mihomo/Clash is working perfectly on your Pi, and every other device using the Pi as their gateway is enjoying that sweet, free internet like a bunch of jolly bunnies. 🐰

Well then - this is the fastest way to reconnect your **PC browser** to the VPN.  
(It has its limits though - it'll only work for your browser, not the whole system.)

---

### 🎯 What We Want to Achieve

Send all Firefox traffic (and DNS lookups) through:

* **SOCKS5 Proxy**
* At IP: `<Pi_IP>`
* Port: `7890` (default Mihomo mixed port)

---

### ✅ EXACT Settings to Fill

![firefox_network_setting](/assets/img/posts/firefox_network_setting.png)

| Field                               | Value                            |
| ----------------------------------- | -------------------------------- |
| **SOCKS Host**                      | `<Pi_IP>`                |
| **Port**                            | `7890`                           |
| **SOCKS v5**                        | ✅ Check this                     |
| **Proxy DNS when using SOCKS v5**   | ✅ **Check this** (important!)    |
| Leave HTTP/HTTPS Proxy fields blank | ❌ do not fill                    |
| **No proxy for**                    | `localhost, 127.0.0.1` |

You DO NOT need to check "Use this proxy for HTTPS" or fill HTTP/HTTPS fields - you're using SOCKS only.

---

### 🧪 To Confirm It Works

1. Open Firefox
2. Visit: [https://api.myip.com](https://api.myip.com)
   → You should see **Node IP** 🌎
3. Go to: [https://browserleaks.com/dns](https://browserleaks.com/dns)
   → You should **NOT** see local ISP DNS - only foreign ones

---

If it doesn't work:

* Make sure your Mihomo is actually **running** on the Pi
* Confirm `allow-lan: true` is in the config
* Confirm `mixed-port: 7890` is open on all interfaces (`0.0.0.0`)
* From your PC, try:

```bash
curl --proxy socks5h://<Pi_IP>:7890 https://api.myip.com
```

---

## 🍓 Shut up - Pi!

We are gonna **reset all network settings** and **turn off your Pi setup** without breaking anything.

This would **Temporarily cut all the complexity** and get your PC back to a fresh, clean internet state (no proxy, no strange gateway routes, no DNS tweaks).

---

### ✅ 1. **Turn Off Pi First**

Just SSH in and:

```bash
sudo shutdown now
```

Or unplug it directly if you're sure it's safe.

---

### ✅ 2. **Reset PC Network Config to Default**

Make sure your router **isn't still using the Pi's static IP** as its DNS.

If it is, *log into your router's settings* and change the DNS to something else.

Then you can delete all saved configs:

```bash
sudo rm /etc/NetworkManager/system-connections/*
sudo systemctl restart NetworkManager
```

---

### ✅ 3. **Make Sure These Are Back to Defaults**

* **IPv4 Method**: `Automatic (DHCP)`
* **DNS**: leave **blank** (or toggle "Automatic" on)
* **Gateway**: will auto-fill
* **Proxy in Firefox**: Set to `No Proxy` or `Use system settings`

---

### 🔄 4. (Optional) Reboot

Just to be clean:

```bash
sudo reboot
```

---

### 🧼 What This Resets:

* IP config (static → auto)
* DNS (custom → default)
* Proxy (if using Firefox)
* No more routing through Pi
* Reverts to router → ISP

---

## 😿 Reinstall Clash Verge, Wipe That Pain Away

SOCKS5 Proxy in Firefox only helps Firefox itself - so yeah, it's just an *emergency band-aid*.

Now, you've turned off that Firefox SOCKS5 setting. It's time to **reinstall Clash Verge**.  
Trust me - your pain's about to drift far, far away. (At least you'll be able to open GitHub again… *right?* 🤨)

As for **Clash CLI**?  
Not today. (**Truth: it fails *way* too easily in this condition.**)  
Even Grandma prefers a GUI - and honestly? So do I.

---

> ⚠️ **WARNING:** Make sure you've **backed up** your config file or your subscription link before you wipe anything!

### 1. ❌ Uninstall the current one:

```bash
sudo apt remove clash-verge
sudo apt purge clash-verge
```

💡 *"Purge" deletes config files stored in system paths (but not in `~/.config/`)*

### 2. 🧹 Clean leftover user configs:

```bash
rm -rf ~/.config/clash-verge*
rm -rf ~/.local/share/clash-verge*
```

Optional:

```bash
rm -rf ~/.config/clash/
```

### 3. ✅ Reinstall the latest .deb:

```bash
wget https://github.com/clash-verge-rev/clash-verge-rev/releases/download/v2.2.3/Clash.Verge_2.2.3_amd64.deb
sudo apt install ./Clash.Verge_2.2.3_amd64.deb
```

Then launch it:

```bash
clash-verge
```

---

## ✅ After Reinstalling - Setting Example

* Reimport your config or subscribe again
* Toggle something on like this setting example

![Clash Verge Setting](/assets/img/posts/clash_verge_1.png)

### ⛓️ TUN

Enable TUN, check if there's any tips showed up to install TUN service. Make sure it is properly downloaded.

![TUN](/assets/img/posts/TUN.png)

After enabling TUN:

```bash
sudo ip route flush table main
sudo systemctl restart NetworkManager
```

Optional debug:

```bash
ip route show table main
```

### 💻 DNS Overwrite

Try this, harmless, fr 🌻

```yaml
enable: true
enhanced-mode: fake-ip
fake-ip-filter:
- '*.lan'
- '*.local'
- '*.arpa'
- localhost.ptlogin2.qq.com
- msftncsi.com
- www.msftconnecttest.com
fake-ip-range: 198.18.0.1/16
fallback:
- https://1.1.1.1/dns-query
- https://dns.google/dns-query
listen: :53
nameserver:
- 8.8.8.8
- 1.1.1.1
- https://dns.google/dns-query
- https://cloudflare-dns.com/dns-query

```

👉 **DO NOT** use any Pi IP for DNS inside Clash, or it will break when Pi is off.

---

#### 🪦 Disable IPv6 (important)

IPv6 can potentially leak your DNS.

Disable IPv6 globally — make your OS and router forget it ever existed:

```bash
# For Linux (temporary)
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1

# To make it permanent:
echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Make sure to check under `Settings` ➡️ `Network` and see if `IPv6` is deactivated.

> Don’t forget to disable/remove IPv6 on your **phone** as well — and make sure you're only using your safe DNS servers.

---

### 🚪 Port Config:

1. Go to `Port Config` section.
2. Click the blue "Random Port" icon once - that will unlock it temporarily.
3. They are editable now, **type in the following** (a classic combo):

   ```bash
   Mixed: 7890
   HTTP: 7891
   Socks: 7892
   Redir: 7893
   TProxy: 7894
   ```

    ![clash_verge_port_config_0](/assets/img/posts/clash_verge_port_config_0.png)

4. **Click SAVE**. 🧿
5. Restart the core (or restart Clash Verge if needed).

### 🖤 TL;DR Cheat Sheet

| Feature       | Should Be...    | Why                               |
| ------------- | --------------- | --------------------------------- |
| TUN Mode      | ON              | Reroutes all traffic              |
| System Proxy  | OFF             | No conflict with TUN              |
| DNS Overwrite | ON              | Prevents leak + DNS injection |
| Port Config   | **Static 789x** | Let you use curl/scripts reliably |
| Firefox       | No Proxy        | TUN mode handles everything       |

---

## 🧠 Final Troubleshoot!

Make sure you can successfully select a node - no more "all timeout"!

*You're doing so good right now!* 😼 Try:

```bash
curl --proxy http://127.0.0.1:7890 https://api.myip.com
curl https://api.myip.com  # Without proxy, test real route
```

If they show different IPs, proxy working.

But if only shows node IP, you may already have *phree* Internet now.

---

### 😾 Clash is not Listening the Right Port 7890

If you met condition like this:

* `curl --proxy http://127.0.0.1:7890` → **connection refused**

```bash
> curl --proxy http://127.0.0.1:7890 https://api.myip.com
{"ip":"<Node_IP>","country":"<Node_Country>","cc":"<Node_Country>"}%
curl https://api.myip.com
curl: (7) Failed to connect to 127.0.0.1 port 7890 after 0 ms: Connection refused

```

This happens probably cause Clash is not listening the right port 7890, but on other port.

Let's try:

```bash
lsof -i -P -n | grep LISTEN | grep clash
```

If it shows something like this:

```sql
clash-ver 10693 $USERNAME   21u  IPv4  82026      0t0  TCP 127.0.0.1:33331 (LISTEN)
```

then we know:

| Field             | Meaning                                                                   |
| ----------------- | ------------------------------------------------------------------------- |
| `clash-ver`       | The binary running - your **Clash Verge core** ✅                          |
| `10693`           | The PID of your Clash core process                                        |
| `127.0.0.1:33331` | The port it's **currently** listening on for **controller/web API stuff** |
| `(LISTEN)`        | It's active and receiving connections 🧩                                  |

---

#### 🤔 Possible Solution

This may affect tools like `git`, `curl` stuff, and others. Normally, we'd set:

```bash
http.proxy=http://127.0.0.1:7890
https.proxy=http://127.0.0.1:7890
```

in Git config.

But in this case, the proxy **isn't** running on `127.0.0.1:7890`.

(Yeah - even if you explicitly set the `Mixed Port` to `7890`, it might not apply immediately.)

Instead, it could be using **a different port** - like that `33331` I showed you earlier using `lsof`.

If you try to install a library with `pip`, you might get something like:

`Failed to establish a new connection: [Errno 111] Connection refused`

But you might think:

*"Why not just update Git to use the correct (but kinda weird) port?"*

Okay, let's do that:

```bash
git config --global http.proxy http://127.0.0.1:33331
git config --global https.proxy http://127.0.0.1:33331
```

Double check it worked:

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```

You should see:

```bash
http://127.0.0.1:33331
http://127.0.0.1:33331
```

Sure, that works for Git - mostly.

But `pip`? Yeah… it still probably won't work.

And just in case the weird port changes again for some unknown reason...

Don't worry - I've got a **better way** right here. 👇🏻

---

### 💡 Solve Port Issues

So now, try to toggle **both** the HTTP(S) and SOCKS proxy ports to "on":

![http(s)/socks ports](/assets/img/posts/clash_verge_port_config.png)

Then open your shell config:

```bash
nano ~/zshrc
```

or, if you're using Bash:

```bash
nano ~/bashrc
```

Add the following lines at the end of the file:

```bash
# 🌩️ Clash Proxy for tools like curl/pip
export http_proxy="http://127.0.0.1:7892"
export https_proxy="http://127.0.0.1:7892"
export all_proxy="socks5://127.0.0.1:7891"

# 🧙 Git Proxy Toggles
# Add more if you use other proxy clients
alias git-proxy-clash='git config --global http.proxy http://127.0.0.1:7892 && git config --global https.proxy http://127.0.0.1:7892'
alias git-proxy-off='git config --global --unset http.proxy && git config --global --unset https.proxy'

# 🔁 Shell-wide Proxy Aliases
# Add more if you use other proxy clients
alias proxy-on-clash='export http_proxy="http://127.0.0.1:7892"; export https_proxy=$http_proxy; export all_proxy="socks5://127.0.0.1:7892"'
alias proxy-off='unset http_proxy https_proxy all_proxy'

# 💡 Check current status
alias proxy-status='echo "🌐 http_proxy: $http_proxy"; echo "🧊 https_proxy: $https_proxy"; echo "🕳️ all_proxy: $all_proxy"'
```

Now try `pip` again, and HELL YEA! That's what we are talking about! 😾

#### 🧠 Understand `git config` & `export http_proxy` & `export all_proxy`

##### 1️⃣ The **`git config`** ones (e.g. `git-proxy-clash`)

👉 They **only affect Git**, and they **persist globally** (across reboots/shells) because they directly modify your Git config:

```bash
git config --global http.proxy ...
```

They are **stored in `~/.gitconfig`**, so even if you reboot or close your terminal, Git will still use the proxy until you `unset`.

💡 **Use this when:**

* You're having trouble pushing/pulling from GitHub or GitLab behind internet restrictions
* You want Git to always use a specific proxy no matter the shell or script

---

##### 2️⃣ The **`export http_proxy`** ones (e.g. `proxy-on`)

👉 These affect **the whole shell environment**, and work with any CLI tool that respects `http_proxy`, `https_proxy`, and `all_proxy`, like:

* `pip`
* `wget`, `curl`
* `npm`
* `apt`, etc.
  ⚠️ But they **only last for the current terminal session**, unless added to `~/.zshrc`.

💡 **Use this when:**

* You want tools like `pip`, `curl`, `npm` to go through your proxy
* You're temporarily testing something in a shell session

---

##### 3️⃣ The **`export all_proxy`** ones (e.g. `proxy-on`)
  
```bash
export all_proxy="socks5://127.0.0.1:7892"
```

This means:

➡️ "**Any program that understands `all_proxy`** (like `curl`, `apt`, or some Python scripts) should send *all* network traffic through a **SOCKS5 proxy** at `127.0.0.1:7892`."

###### 🔧 Tools That Expect SOCKS?

These are apps or environments that read `all_proxy` and route **all protocols** (TCP/UDP) via SOCKS-some don't care if it's HTTP or SOCKS, they just want to tunnel. Typical examples include:

* `curl` (if you specify `--socks5`)
* `apt` (if explicitly configured to use SOCKS)
* `python requests` if you set `all_proxy`
* `conda`, `npm`, and even **some Electron-based apps**
* 🧙‍♀️ **Git** (in some cases) - mostly prefers `http_proxy`, but fallback is possible

---

### 🤔 Should you keep this block?

#### 🧩 What's happening now:

* ✅ `Mixed Port` (7890): **for GUI, internal core, or apps like Clash Dashboard**
* ✅ `HTTP(S) Port` (7892): **for pip, curl, git**
* ✅ `SOCKS Port` (7891): **for tools supporting SOCKS5 or when using `all_proxy`**

Once you found:

```
verge-mih 17100 root ... TCP localhost:7890 (LISTEN)
```

That means Clash Verge's **core is alive and well**. Your whole kingdom is back online 🏰.

---

#### 💡 Then should you still keep this?

```bash
# 🌩  Clash Proxy for tools like curl/pip
export http_proxy="http://127.0.0.1:7892"
export https_proxy="http://127.0.0.1:7892"
export all_proxy="socks5://127.0.0.1:7891"
```

**YES!** This is *golden* if:

* You wanna install packages (`pip`, `npm`, `curl`, etc.) over internet restrictions
* You have `SOCKS` toggle on ✅
* You're switching tools and protocols without confusion

🔧 *Extra note:* if the SOCKS toggle is off, the `all_proxy=socks5://127.0.0.1:7891` would silently fail for tools that try SOCKS only (like some weird `apt` or IRC clients). So **yes**, keep both 7891 + 7892 toggled **on** if you're setting all three proxy envs.

## 🐈 Final Thoughts for Pi + Clash

Pi does work great with Clash CLI - no doubt. But if you:

* don't want your Pi messing with your PC's DNS  
* like to frequently tinker with proxy clients on your PC  
* prefer more control and flexibility in your network setup  

Then maybe just leave your Pi's *phree* internet for your other devices - like your phone, iPad, etc.

And seriously - **don't use the Pi's DNS on your PC**.  
Just activate your proxy directly on the PC, and go wild with your own rules and configs. ⚡

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
