---
layout: post
title: "From Clash to V2RayN: Unlocking SOCKS5"
description: "A smooth walkthrough switching from Clash to V2RayN on Linux: fixing port conflicts, bypassing ads on Spotify, and fine-tuning SOCKS5 proxy magic that just works."
date: 2025-06-02 01:47:00 +0800
categories: [🤖 tech, 🔒 Web Security]
tags: [🐧 Linux, 💻 Networking, 😼 Clash, 🌀 VPN, 🧦 V2Ray, 🌀 SOCKS5, 🚪 Port, 🧙 Git, 🔁 HTTP/HTTPS Proxy, 🕳️ SOCKS5, 🐞 Linux troubleshooting, 🎧 Spotify, 🖥️ CLI]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/clash_v2ray.png
---

So recently, Clash has been performing well based on [this setup](https://kay-a11y.github.io/posts/troubleshoot-clash-port/).

But I ran into an issue when switching to V2Ray.  
(**v2rayN on Pop_OS**, if that matters.)  
I could connect to my node just fine on my phone - but not on my PC.  
And when I tested all node delays in v2rayN, they all looked normal and green.

So, based on the previous setup, I'm gonna troubleshoot a few key points  
to make sure we can switch to V2Ray *smoothly.*

---

## 🚪 Proxy port setup

Check out your V2Ray mixed port(normally it should be 10808).

Edit your `~/.bashrc` or `~/.zshrc` and Comment the proxy setup for Clash, add one for v2rayN `export all_proxy="socks5://127.0.0.1:10808"`.

```bash
# 🌩  Clash Proxy for tools like curl/pip
#export http_proxy="http://127.0.0.1:7892"
#export https_proxy="http://127.0.0.1:7892"
#export all_proxy="socks5://127.0.0.1:7891"

# 🌩  v2rayN Proxy for tools like curl/pip
export all_proxy="socks5://127.0.0.1:10808"
```

---

Use this alias if you've used [my previous setup](https://kay-a11y.github.io/posts/troubleshoot-clash-port/#-solve-port-issues).

```bash
git-proxy-v2ray
```

Or if not:

```bash
git config --global http.proxy http://127.0.0.1:10808 && git config --global https.proxy http://127.0.0.1:10808
```

---

Then reload:

```bash
source ~/.zshrc
```

or

```bash
source ~/.bashrc
```

🧪 Test with:

```bash
curl -x socks5h://127.0.0.1:10808 https://api.myip.com
```

Failed with `error code: 1005`? Check if [this part](#-final-test-sudo-reboot--launch-v2rayn--use-socks) helps!

---

## 🦊 Make Firefox obey proxy

Go to:

> **Firefox → Settings → Network Settings → Manual Proxy Configuration**
> Set:

* SOCKS Host: `127.0.0.1`
* Port: `10808`
* No proxy for: `localhost, 127.0.0.1`
* Check "Proxy DNS when using SOCKS v5"

![firefox_network_setting_v2ray](/assets/img/posts/firefox_network_setting_v2ray.png)

Now test your IP & DNS servers with [BrowserLeaks](https://browserleaks.com/dns) - it should work instantly.

---

## 💢 TUN Conflict Check (if you used Clash before)

Check if there's leftover `tun0` or custom route:

```bash
ip a
ip r
```

If you see `198.18.x.x`, `fake-ip`, or `utun`, kill them:

```bash
sudo ip link delete tun0
sudo ip rule flush
sudo ip route flush table main
sudo systemctl restart NetworkManager
```

---

## 🧹 Lock in Clean DNS

If you're running `systemd-resolved`, and your DNS is:

```text
nameserver 127.0.0.53  ← stub resolver (systemd-resolved)
```

Yet behind the scenes, it's actually forwarding to router,

for example, static:

```text
1.1.1.1
8.8.8.8
```

Now Just change `/etc/resolv.conf` (keep systemd-resolved)

```bash
sudo rm /etc/resolv.conf # Remove symlinked `resolv.conf`
echo -e "nameserver 1.1.1.1\nnameserver 8.8.8.8" | sudo tee /etc/resolv.conf # Create static one
sudo chattr +i /etc/resolv.conf # Make it immutable (no more random override!)
```

> Change `1.1.1.1` and `8.8.8.8` to your router DNS.

### 🔄 if you ever wanna revert:

```bash
sudo chattr -i /etc/resolv.conf
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

This is totally safe and revertible. This way you **skip the stub resolver**, send DNS **direct to your router's DNS servers (e.g. Cloudflare/Google**).

---

### ❗ About the 3 lines in `/etc/resolv.conf`

| Line                     | Meaning                                                               |
| ------------------------ | --------------------------------------------------------------------- |
| `nameserver 127.0.0.53`  | Stub resolver - systemd-resolved interception                         |
| `options edns0 trust-ad` | Extended DNS flags - fine to keep or ignore                           |
| `search .`               | Domain search - totally safe to remove or leave, it does nothing here |

**You only really need to change the `nameserver`** to your desired DNS.

---

## 🧪 Final test: `sudo reboot` + launch v2rayN + use SOCKS

```bash
curl -x socks5h://127.0.0.1:10808 https://api.myip.com
```

Get `error code: 1005`?

Try these:

```bash
curl -x socks5h://127.0.0.1:10808 https://httpbin.org/ip

# Set Custom User-Agent
curl -A "Mozilla/5.0" -x socks5h://127.0.0.1:10808 https://icanhazip.com

# Try one of these known good APIs:
curl -x socks5h://127.0.0.1:10808 https://ifconfig.me
curl -x socks5h://127.0.0.1:10808 https://ipinfo.io/ip
curl -x socks5h://127.0.0.1:10808 https://icanhazip.com
curl -x socks5h://127.0.0.1:10808 https://wtfismyip.com/text

```

### 🔍 What Is Cloudflare `error 1005`?

That means:

> 🔒 **Access Denied - Your IP has been flagged** (by WAF, rate-limits, or country ban)

Cloudflare does this:

* If the IP is in a **VPN range**
* If the proxy IP is **suspicious** (many access requests)
* If your TLS signature, headers, or behavior looks **bot-like**
* Or if the site owner blocks entire countries

**api.myip.com** uses Cloudflare, and your **node's IP** is probably on a list if you got `error code: 1005`.

---

## 🧠 All set! What's Working *Flawlessly* Now (Based on my test)

| App                           | Result                  |
| ----------------------------- | ----------------------- |
| **Discord**                   | ✅ YES                   |
| **Spotify**(flatpak)          | ✅ YES (No Proxy)         |
| **Chrome**                    | ✅ YES                   | 
| **Telegram**                  | ✅ YES (needs SOCKS5 manually configured, similar with [firefox's](#-make-firefox-obey-proxy)) |
| **VSCode Git**                | ✅ YES                   |
| **HexChat**                   | ✅ YES                   |
| **Firefox**                   | 😐 Set manually or `use system proxy settings` |

---

## 🎵 Here's the Spicy one - About Spotify ads

So far, I've found two actually useful ways to block Spotify ads.

* [Spotify Web](https://open.spotify.com/) + **uBlock Origin**  
* **Spotify Desktop**: Route it through your **V2Ray proxy** - exactly what we set up earlier!  

> ✅ **Spotify Desktop = NO ads**, but *only* when it's running through **V2RayN + private & clean nodes**.  
> ❌ **Clash Verge + Rule mode**? That doesn't phreakin' work.

What might be going on inside SOCKS5?  
Honestly, I'd *love* to know...

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
