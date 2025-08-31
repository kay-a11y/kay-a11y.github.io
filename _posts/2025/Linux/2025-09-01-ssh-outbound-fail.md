---
layout: post
title: "[FIX] Broken SSH & Proxy, SO AM I!"
description:
date: 2025-08-31 19:19:53 +0000
categories: [🤖 tech, 🐧 linux]
tags: [🐧 Linux, 🖥️ CLI, 🔀 LAN]
img_path: /assets/img/posts/
math: true
toc: true
comments: true
image:
---

I finally tasted just how fragile the system gets when every single proxy drops dead, and the firewall sometimes even entirely drop the 443. Somewhere out here, people are trapped in this suffocating local area network, drowning in fake comfort and plastic smiles. But we should have coded for open networks, born for raw freedom. That's the pulse we live by.
And no, we sure as hell do not forget nor do we forgive.

## TL;DR

**Cloudflare Zero Trust**/**WARP** dropped a whole nftables **output** chain that only allowed 53/80/443, which starved SSH and most nodes. After reboot it re-armed itself. To fix: purging WARP + flushing nft/iptables + resetting UFW to allow outgoing SSH.

## Background

This post is for troubleshooting a broken SSH session, where outbound just suffocates, and every port but 443 is choked out.  

A few days ago I updated my Pop!_OS. Right after it finished, something I'd almost forgotten about, `Cloudflare Zero Trust` suddenly decided to boot itself up. I barely ever used it, so I just ignored it, and went for a full reboot to apply the update.  

Then came the disaster:  

1. Every single node on Clash started timing out. v2ray nodes dropped dead, except the one hiding on 443, but even that one turned unstable after reboot. Plus if the firewall just drop down the port, that node would immediately die too.  
2. When I tried SSH into my VPS, commands would just hang like forever, only to end in `time out` or `Could not resolve hostname... Name or service not known`.  

Tbh, I kinda started to afraid of doing `sudo apt upgrade` again.  

What still works:

1. SSH to my EC2 via browser Instance Connect
2. **Inbound SSH works**. (Pop listens on port 22 and another computer connects fine. Try SSH to pi: `ssh pi@192.168.1.200`)
3. DNS and routing are functional
    > Check if DNS and routing are functional:
    >
    > ```bash
    > # Replace HOST with your VPS DNS
    > HOST=ubuntu@<IP>
    > 
    > getent hosts "$HOST"
    > dig +short "$HOST"
    > dig @1.1.1.1 +short "$HOST"     # bypass local resolver
    > ```
    >
    > If `dig @1.1.1.1` returns an IP but the others don't, your local DNS is borked (Clash/TUN or systemd-resolved).
    >

---

## Troubleshoot

### 1. Check MTU / Packet Size Limits

Run this to see what's happening with large packets:

```bash
ping -M do -s 1472 bandit.labs.overthewire.org
ping -M do -s 1400 bandit.labs.overthewire.org
```

* If the 1472-byte ping fails but the 1400-byte one works, we've got an MTU mismatch.
* If both fail or both succeed, we move to the next probe.

In my condition, both hang, meaning the packets never even leave the NIC. The block is *before* size matters.

---

### 2. Trace the traffic mid-hang

```bash
# find your active interface, eg `enp3s0` or `wlan0`
ip route | awk '/default/ {print $5; exit}'
```

BIG thanks for [bandit](https://overthewire.org/wargames/bandit/). If you do not have available VPS yet, try with bandit's ip `56.228.72.241`. And get bandit's authentication password on their site later!

Open one terminal:

```bash
# assume the active interface is `enp3s0`
# sudo apt update && sudo apt install -y tcpdump
sudo tcpdump -ni enp3s0 host 56.228.72.241 and tcp
```

In another, start your SSH:

```bash
ssh -vvv bandit0@bandit.labs.overthewire.org -p 2220
```

Watch if packets are flowing both ways:

* **Seeing both sides** but hang: keepalive or conntrack issue.
* **Only your side**: mid-path drop.

Example output:

```bash
> sudo tcpdump -ni enp3s0 host 56.228.72.241 and tcp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on enp3s0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
# hangs

> ssh -vvv bandit0@bandit.labs.overthewire.org -p 2220
OpenSSH_8.9p1 Ubuntu-3ubuntu0.13, OpenSSL 3.0.2 15 Mar 2022
debug1: Reading configuration data /home/phruit/.ssh/config
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 19: include /etc/ssh/ssh_config.d/*.conf matched no files
debug1: /etc/ssh/ssh_config line 21: Applying options for *
debug3: expanded UserKnownHostsFile '~/.ssh/known_hosts' -> '/home/phruit/.ssh/known_hosts'
debug3: expanded UserKnownHostsFile '~/.ssh/known_hosts2' -> '/home/phruit/.ssh/known_hosts2'
debug2: resolving "bandit.labs.overthewire.org" port 2220
debug3: resolve_host: lookup bandit.labs.overthewire.org:2220
debug3: ssh_connect_direct: entering
debug1: Connecting to bandit.labs.overthewire.org [56.228.72.241] port 2220.
debug3: set_sock_tos: set socket 3 IP_TOS 0x10
# hangs for very very long here
debug1: connect to address 56.228.72.241 port 2220: Connection timed out
ssh: connect to host bandit.labs.overthewire.org port 2220: Connection timed out
```

Confirmed the kernel is not emitting SYNs at all.

---

### 3. Enable SSH keepalives

Add to `~/.ssh/config` (Create one if you don't have this file):

```txt
Host *
  ServerAliveInterval 15
  ServerAliveCountMax 4
```

This only helps once SSH is already established.

---

### 4. Check and clear connection tracking

Sometimes conntrack tables get full and stall packets:

```bash
sudo apt install conntrack -y
sudo conntrack -L | head
sudo conntrack -F
```

That flushes all tracked connections. Then test SSH again.

Example:

```bash
> sudo conntrack -L | head
udp      17 19 src=192.168.1.100 dst=1.1.1.1 sport=58651 dport=53 src=1.1.1.1 dst=192.168.1.100 sport=53 dport=58651 mark=0 use=1
udp      17 14 src=192.168.1.100 dst=1.1.1.1 sport=36733 dport=53 src=1.1.1.1 dst=192.168.1.100 sport=53 dport=36733 mark=0 use=1
tcp      6 89 TIME_WAIT src=127.0.0.1 dst=127.0.0.1 sport=49068 dport=5600 src=127.0.0.1 dst=127.0.0.1 sport=5600 dport=49068 [ASSURED] mark=0 use=1
#...
> sudo conntrack -F
conntrack v1.4.6 (conntrack-tools): connection tracking table has been emptied.
> ssh bandit0@bandit.labs.overthewire.org -p 2220
# hangs
```

---

### Check firewall/policy

Reboot may have re-applied some rules.

#### 1. Dump nftables ruleset

```bash
sudo nft list ruleset
```

See if anything reappeared. Especially look for `drop`/`reject` under `output` or `forward`.

I finally found the culprit here. My nftables has a **`table inet cloudflare-warp`** that installs a base **`output`** chain with **`policy drop`** and only allows a few destinations/ports (53/80/443 and some Cloudflare IPs). Everything else including SSH ports 22/2220 gets dropped.

From the nftables:

```txt
table inet cloudflare-warp {
  ...
  chain output {
    type filter hook output priority filter; policy drop;
    ...
    udp dport 53 accept
    tcp dport 80 accept
    tcp dport 443 accept
  }
}
```

That exactly matches web works, DNS works, **all outbound SSH time out**.

#### 2. Dump iptables

```bash
sudo iptables -S
sudo iptables -t nat -S
sudo iptables -t mangle -S
```

---

## Safest fix: remove WARP completely

1. See what's installed / running

    ```bash
    dpkg -l | egrep -i 'cloudflare|warp'
    systemctl status warp-svc  # if present
    warp-cli status 2>/dev/null || true
    ```

2. Gracefully shut WARP down (if present)

    ```bash
    # turn off any enforced "always-on"
    warp-cli disable-always-on 2>/dev/null || true
    warp-cli disconnect 2>/dev/null || true
    warp-cli delete 2>/dev/null || true

    # stop the service so it can't reapply nft rules
    sudo systemctl stop warp-svc 2>/dev/null || true
    sudo systemctl disable warp-svc 2>/dev/null || true
    sudo systemctl mask warp-svc 2>/dev/null || true
    ```

3. Uninstall WARP (Zero Trust client)

    ```bash
    sudo apt purge -y cloudflare-warp || true
    sudo apt autoremove -y
    ```

4. Remove any lingering firewall config from it

    ```bash
    # nuke any cloudflare nft tables if they're still around
    sudo nft list tables | sed -n '1,200p'
    # if you still see "cloudflare-warp", then:
    sudo nft delete table inet cloudflare-warp 2>/dev/null || true

    # hard reset packet filters
    sudo nft flush ruleset
    sudo iptables -F
    sudo iptables -t nat -F
    sudo iptables -t mangle -F
    ```

5. Put your firewall back in a clean, friendly state

    ```bash
    sudo ufw reset
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow 22/tcp
    sudo ufw enable
    ```

6. Quick connectivity probes (no proxies)

    ```bash
    env -i TERM=$TERM nc -vz portquiz.net 22
    env -i TERM=$TERM nc -vz portquiz.net 2220
    ```

    You should see a quick "succeeded" or at least a prompt, if those succeed. e.g.:

    ```bash
    > env -i TERM=$TERM nc -vz portquiz.net 22
    Connection to portquiz.net (35.180.139.74) 22 port [tcp/ssh] succeeded!
    > env -i TERM=$TERM nc -vz portquiz.net 2220
    Connection to portquiz.net (35.180.139.74) 2220 port [tcp/*] succeeded!
    ```

7. Try SSH again

    ```bash
    env -i TERM=$TERM ssh -vvv bandit0@bandit.labs.overthewire.org -p 2220
    env -i TERM=$TERM ssh -vvv <your_vps>@<vps_ip>
    ```

---

*💓 owed to [Bandit](https://overthewire.org/wargames/bandit/).*

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
