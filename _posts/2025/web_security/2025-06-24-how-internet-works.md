---
layout: post
title: "How the Internet Works: CGNAT, IPv6, Jump Boxes, Remote Access, and Beyond"
description: "This deep-dive covers NAT, CGNAT, dynamic and shared public IPs, AWS jump boxes, BGP/IXP routing, TCP quirks, censorship, DNS/HTTPS privacy, and practical remote access hacks." 
date: 2025-06-25 02:11:00 +0800
categories: [🤖 tech, 🔒 Web Security]
tags: [🐧 Linux, 💻 Networking, 🛡️ CGNAT, 🚇 SSH, 😨 IPv6, 🔄 NAT Routing, 🏠 HomeLab, 🤖 Cloud, 🧑‍💻 BGP, 💡 IXP]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: 
---

## NAT

> **NAT makes it very hard for devices with private addresses to provide services on the internet because they're not easily reachable from the outside.**

Let's simplify:

You've got a **home network**-your laptop, your phone, maybe even a smart toaster. All of them have **private IP addresses**, like:

```
192.168.1.3 (your laptop)
192.168.1.4 (your phone)
192.168.1.5 (your toaster)
```

But the outside world (the Internet!) doesn't understand private IPs. So when any of your devices want to **access the internet**, they **send their packets to the router**, and your **router does two jobs**:

---

NAT = **Network Address Translation**

Your router IS like a very enthusiastic **doorman with sticky notes**:

1. You (laptop) say: *"I want to visit catmemes.com!"*
2. The router:

   * Takes your packet
   * Changes its "From" address to the router's **public IP** (like `12.222.9.5`)
   * Sends it out with a sticky note: "Psst, this was from XX's laptop, I'll remember that!"
3. When the reply comes back from the internet, it:

   * Arrives addressed to `12.222.9.5`
   * The router checks its sticky notes, sees: "Oh! That's for XX's laptop!"
   * Rewrites it back and gives it to you

So all your devices **share that one public IP**, thanks to NAT.

---

### People can't reach *you* in NAT

Because NAT is a one-way door unless you set it up specially.

If someone on the outside says:

> "Hey, I want to talk to your laptop directly!"

The router says:

> "Sorry, I don't know who you're trying to reach inside. That packet's going in the trash."

This is because your **devices don't have public IPs**. The **router didn't expect this conversation**, so it has **no sticky notes to route it**. It **protects you**, but also **blocks you from hosting services**.

If you try to **host a website** on your laptop, **run a Minecraft server**, or even make a **reverse SSH connection**, no one on the internet can find your device, because NAT **doesn't let packets in** without permission.

---

### But there's also some solutions

1. **Port Forwarding**: Tell your router: "If anyone asks for port 22 (SSH), send it to my laptop."
2. **UPnP** (automatic forwarding, less secure)
3. **Reverse Tunnel**: Your device makes the connection outward and **keeps it open**, like:

   ```bash
   ssh -R 2222:localhost:22 you@yourVPS
   ```
4. **Use a VPS** as your public-facing front door and just tunnel traffic to your private home.

---

## IPv4 & Shared IPs

IPv4 is limited. There are only about **4.3 billion IPv4 addresses** in the entire pool. With almost 10 billion devices online? **We're reusing IPv4s everywhere.** But we only **reuse IPv4 addresses in private spaces.**
**Public IPv4s** are still globally unique.

If you're behind NAT, your **internal/private IP** (like `192.168.1.123` or `10.0.0.5`) is **probably the same** as your neighbor or that café Wi-Fi. But these private IPs are scoped **only to your LAN**. They are **not visible** to the public internet.

That *public* IPv4 - the one your **router** uses to connect to the internet - is **unique**, **assigned by your ISP**, and **shared among nobody else** *(unless you're in CGNAT, but we'll explain that too)*.

So how does SSH know where to go? When you SSH like this:

```bash
ssh user@123.45.67.89
```

Your packet is delivered to **that exact public IP** - globally unique. Then, if that IP belongs to a **router**, it'll check if port **22 is forwarded** to someone inside, like your laptop.

Example:

* Public IP: `123.45.67.89`
* NAT rule: port 22 ➝ `192.168.1.123`

What if you're on **CGNAT** (Carrier-Grade NAT)?

In this case:

* Your ISP assigns **you and others** the **same public IPv4**
* You're behind **a second layer of NAT**
* You can **make outbound connections**, but **you can't accept inbound ones easily**
* This is why **many mobile networks**, apartment ISPs, or campus Wi-Fi won't let you host a server - you don't have a **true public IP**

> In CGNAT, SSH into your home PC = Impossible *unless* you use a VPS as a relay (like with reverse SSH)

> I wrote another Post about how to impelement reverse SSH [here](https://kay-a11y.github.io/posts/reverse-ssh/){:target="_blank"}.

---

## CGNAT

1. NAT = **1 layer**

    Your device (e.g., laptop `192.168.1.123`) ➜ router ➜ public IP `123.45.67.89`

    So far so good. One layer of NAT = **manageable**.
    You can:

    * Port forward
    * Reverse SSH
    * Use UPnP
    * Scan your public IP and get results

2. CGNAT = **2 layers (double NAT)**

    You now share your "public" IP with others via your ISP.
    So it's:

    ```txt
    You: 192.168.1.123 ➜  
    Router: 10.10.10.2 ➜  
    ISP gateway: 100.64.0.1 ➜  
    Public IP: 123.45.67.89
    ```

    Your router does NAT, then the **ISP does another NAT**. At this point, direct port forwarding (from outside) to your home PC: NO. You're locked *two layers deep*. You can still *browse*, but you can't *host*. 

    BUT, Reverse SSH (outbound tunnel) is unstoppable, as long as you have outbound access.

3. **Multi-NAT (Triple or More NAT)**

    ```txt
    Your PC: 192.168.1.123
    → Your Home Router: 10.0.0.2
    → Community Gateway Router: 172.16.1.1
    → ISP CGNAT Router: 100.64.1.0
    → Real Internet: 42.13.20.99
    ```

    That's **three layers of NAT**. That's **multi-NAT**. That's **network purgatory**. No Port forwarding. Won't reach UPnP.

    But Reverse SSH can stil work through multi-NAT (3+ layers of NAT). As long as **each NAT layer allows outbound connections to the internet** (especially outbound TCP/SSH), **you can punch through** as many layers as you want using reverse SSH.

    But if **any** router or NAT in the chain blocks *outbound* SSH (or your VPS port), it won't work. Sometimes, "captive portals" (like in hotels or locked WiFi) might block outbound SSH or limit ports. But **most home, ISP, and even many campus/corporate NATs** allow outbound SSH by default.

---

### How to confirm CGNAT?

1. Check the **WAN IP** on your router admin interface

    If you see `10.x.x.x` WAN IP? That's private IP space. Your router **thinks it's on the internet**, but it's **not directly exposed** - it's behind another upstream NAT = ***CGNAT*** confirmed. → **Double NAT = CGNAT**

2. **Trace Your Path with Traceroute**

    ```bash
    sudo apt install traceroute
    traceroute 1.1.1.1
    ```

    Or:

    ```bash
    traceroute 8.8.8.8
    ```

    Look at the first few hops:

    * 1st hop = your router
    * 2nd hop = real public IP → **Single NAT**
    * 2nd hop = 10.x.x.x / 100.64.x.x → **Double NAT or CGNAT**
    * 2nd + 3rd = both private IPs → **Multi-NAT**

---

### Dynamic Public IP

> If your public IP changes sometimes, like from `12.234.56.166` to `.196`

That's a **dynamic public IP from a shared pool**, **not bound** to your modem/router MAC.

In normal PPPoE setups (with static IP or direct public assignment), your public IP might stay the same for weeks/months, or **your router's WAN IP = your public IP**.

But in CGNAT:

WAN IP = `10.x.x.x`, with dynamic Public IP, which is a **Classic CGNAT behavior**

---

### You can't port forward under CGNAT

Try this:

* Go into router
* Set port forward rule to your internal IP (e.g. SSH to port 22)
* Go to canyouseeme.org → test open port 22

You'll fail - because your router doesn't have the *real* external IP. Only your ISP does - and unless you pay extra, normally you **can't map ports inbound under CGNAT**.

---

## IPv6

**IPv6 is so huge, we'll never run out.** Everyone and everything could get trillions of unique IPs - your phone, fridge, toaster, robot cat...

Let's calculate:

1. **Total IPv6 addresses:**

    IPv6 uses **128 bits** per address.
    So total possible addresses:

    ```txt
    2^128 = 340,282,366,920,938,463,463,374,607,431,768,211,456  
    (aka ~340 undecillion addresses)
    ```

2. **Earth's surface area**

    Earth's total surface = ~510 million square kilometers. Let's convert that to **square millimeters**:

    ```txt
    510,000,000 km²  
    = 510,000,000 × (1,000,000 m²/km²)  
    = 5.1 × 10^14 m²  
    = 5.1 × 10^14 × (1,000,000 mm²/m²)  
    = 5.1 × 10^20 mm²
    ```

    So Earth has **5.1 × 10²⁰ square millimeters** of surface area.

3. Addresses per mm²

    Now divide IPv6 addresses by Earth's square mm:

    ```txt
    (2^128) / (5.1 × 10^20)
    ≈ 6.65 × 10^17
    = 665,570,793,348,866,944
    ```

    So you get over **665 quadrillion addresses for every single square millimeter** of Earth. Like, even your cat's toe bean could have its own subnet.

---

### Unique Local Address (ULA)

It's like **IPv6's version of private IPs**, like:

* In IPv4: `192.168.x.x`, `10.x.x.x`
* In IPv6: `fc00::/7` (usually `fdxx::` for ULAs)

They're **not reachable from the internet**. Just used **inside your house**, for local stuff like talking to your Raspberry Pi, streaming to your smart TV, Sending you a message across LAN like: `echo "cutie" | nc fd69::1 9999`.

These **can't be reached globally**. Only your internal network sees them.

---

### Global IPv6 Public Address

In IPv4, your devices usually get **one private IP** behind NAT. So your toaster can't be accessed from the outside unless you port forward. But with **IPv6** each device gets its own public, globally routable address.

That means no NAT needed, and your laptop, phone, or even fridge can be reached directly from anywhere on the internet. 

There are **publicly reachable**: These IPs can be used to **host services, servers, anything** - without hacks or tricks.

---

### Can I SSH into my PC from another device?

It *totally depends* on whether your PC is **reachable**, and that depends on **IPv4 vs IPv6**.

1. **Home PC with IPv4 only**

    Your home network = **behind NAT** (like we discussed earlier).

    So Your PC has a private IP like `192.168.1.100`, and your router has a public IP like `123.45.67.89`. You can **SSH to your PC locally** using:

    ```bash
    ssh you@192.168.1.100
    ```

    BUT from *outside* (like from your phone's LTE or a VPS), trying this **won't work unless you do port forwarding** on your router. Because the router doesn't know **which device** to send port 22 traffic to. NAT blocks unsolicited inbound connections by default.


2. Home PC with IPv6 enabled

    If your PC **has IPv6 enabled** AND your ISP + router support it. It gets a **public IPv6 address**, like `2001:0db8:abcd:1234::1`. That means it's **reachable from the whole internet** - no NAT.

    * You can just:

    ```bash
    ssh yourusername@2001:0db8:abcd:1234::1
    ```

    BUT *Firewalls still apply*. Most routers still block incoming IPv6 by default (because it would be wild otherwise), so you'd have to enable inbound connections to port 22 on your router/firewall.

    And maybe set a firewall rule on your PC too:

    ```bash
    # Configure `ufw` to allow SSH on IPv6
    sudo ufw allow 22/tcp
    sudo ufw allow from <your_remote_ipv6> to any port 22
    ```

3. **AWS: Why can you SSH to it via IPv4?**

    AWS **assigns you a public IPv4** - straight up! No NAT, no port forwarding, nothing hidden.

    When you launch an EC2 instance, AWS gives it:

    * A **public IPv4**: like `13.58.123.45`
    * An **internal IP** (for VPC stuff): like `172.31.x.x`

    So you *connect from the outside* using:

    ```bash
    ssh ec2-user@13.58.123.45
    ```

---

### AWS - example of jump box

Inside the AWS VPC, that instance's **true IP** is `172.31.x.x`. The public IP is kind of like a **"mask"** placed *on top of* the private IP.

Under the hood, AWS does something like **1:1 NAT** - called **Elastic IP mapping**. This isn't port forwarding like your home router. Instead, it's more like:

| What                                      | How it works                                         |
| ----------------------------------------- | ---------------------------------------------------- |
| You connect to `13.58.123.45`             | AWS maps that to internal `172.31.x.x` automatically |
| No need for port forwarding               | The mapping is **transparent** and **pre-wired**     |
| Works for **all ports**, not just port 22 | Unlike home NAT, which needs per-port rules          |

```
[You on public internet]
     |
     v
[ 52.193.166.65 ] ← (AWS auto NAT mapping) → [ 172.31.32.78 ]
                                |
                           [EC2 instance]
```

It's a **global NAT layer AWS manages** for you.

It feels like:

```bash
[Internet] → [13.58.123.45] → [172.31.x.x EC2 instance]
```

But AWS doesn't even call it "NAT" in most docs.
They treat the public IP as a property of the instance - but under the hood, **it's being translated into the private IP**, kind of like "cloud-scale NAT with automation."

You can actually SSH into your instance using *both*:

```bash
ssh ec2-user@13.58.123.45   # from internet
ssh ec2-user@172.31.x.x     # only from inside same VPC
```

And both go to the same machine.

*Because All EC2s in the same VPC are like one big LAN.*

They can:

* Ping each other by **private IP**
* SSH into each other via **`172.31.x.x`**
* Exchange traffic **without using the internet**

This is **why** the bastion pattern exists. You're literally **jumping into the VPC** using **one publicly exposed server** and then **traversing privately** inside.

Say you have an EC2 in Tokyo. And you only know another EC2's **private IP** and **private key**, but not its public IP. So your flow would be:

```bash
# On your laptop:
ssh -i Tokyo.pem ec2-user1@12.x.x.x   # SSH into YOUR EC2 (public IP)

# Now you're inside AWS VPC
# On that EC2:
ssh -i his-secret-key.pem ec2-user2@172.11.y.y     # SSH to friend's private EC2
```

IRL, when working in corp networks:

* All internal servers are **private-only**
* You get access to a **jump box**
* From there, you pivot into:

  * Database servers
  * App servers
  * Dev environments
  * Sometimes internal admin panels

---

Thats why AWS says never pentest others' EC2s. **You absolutely should NOT scan, nmap, fuzz, or poke EC2s you don't *own* or *explicitly have permission for*** even if they're in the **same VPC**, on the **same subnet**, sharing a coffee machine with your packets.

But you can totally treat your **home LAN** like your **own personal VPC**.

> Your **home network** = a **private IP space**, usually `192.168.x.x` or `10.x.x.x`  
> All devices on your Wi-Fi/router share that space  
> You have **full control** over it 

Like:

| AWS VPC                | Your Home LAN                    |
| ---------------------- | -------------------------------- |
| Custom IP range (CIDR) | Router assigns 192.168.x.x       |
| EC2 instance           | Your laptop                      |
| Security group         | Your firewall / router config    |
| Public IP (via NAT)    | Your router's external IP        |
| Private IP             | Your local IP (e.g. 192.168.1.6) |
| Jump box               | Your main PC                     |

You can build your mini lab at home. Then you can:

* Run vulnerable web apps like **DVWA**, **Metasploitable**, or **bWAPP** on the laptop
* Scan it with **Nmap** from your PC
* Intercept packets with **Wireshark**
* Exploit it with **Metasploit**, **Nikto**, **Gobuster**, etc
* Write up your own **CVE reports for fun**

---

## Enable IPv4 vs IPv6

> **If you turn off IPv6 and only use IPv4**, then you're always using **NAT** (Network Address Translation), and your **router is rewriting source IPs** for *every* outbound packet.

Your router is constantly:

| Packet Direction       | Rewriting What?    |
| ---------------------- | ------------------ |
| Outbound (you → world) | **Source IP**      |
| Inbound (world → you)  | **Destination IP** |

That's NAT in action.

IPv6 was designed to **get rid of NAT** entirely. Because:

* You get a **real global IP** on every device
* No rewriting needed
* It's **cleaner**, more transparent
* But: **you have to firewall carefully** because you're *directly* on the internet

So with IPv6 **off**, you're living in a NAT'ed bunker. With IPv6 **on**, your device might be wearing a bikini on a public beach.

**No NAT = Direct Traffic**, Potentially this means **FASTER**:

* No NAT overhead means slightly less CPU load on router
* No NAT translation tables = fewer lookups = less delay
* Routing becomes more **transparent and efficient**
* Fewer things to break (like port forwarding bugs)

BUT in real life, the **speed gain is usually small**, unless you're under load or in edge cases (e.g., running a game server, P2P app, or massive download/upload sessions).

---

## BGP (Border Gateway Protocol) - the internet's GPS for routers

> **BGP** routers prefer paths that go through AS's (Autonomous System) own network, rather than asking a neighbor for help, because going through someone else **adds a "hop"**, making it feel longer.

An **Autonomous System (AS)** is like a big island in the ocean of the internet. It's usually:

* An ISP (like Google)
* A university
* A huge company

Each AS has a unique number - like **AS12345**.

---

### What does BGP do?

Routers talk to each other using BGP to say:

> "Yo, if you want to reach **this IP address**, I can get you there in X hops through me."

Then every router builds a map based on these offers, like choosing which subway route to take.

Imagine this:

* You're **AS100**, and you're deciding how to reach IP `8.8.8.8`
* You have **two options**:

| Route                          | Hops | Description                                |
| ------------------------------ | ---- | ------------------------------------------ |
| Through **your own AS**        | 3    | Internal routing, fully in your control |
| Through **AS200** (a neighbor) | 4    | Goes out of your network                |

**BGP will prefer the first route** - even if both paths work.
Because:

* Fewer hops = better
* More control = safer, more predictable
* No need to depend on someone else's network

---

## IXP (Internet Exchange Point)

An **IXP** is like the **train station** where **big internet providers (ASes)** meet and exchange data.

Imagine:

* AS1 = A university
* AS2 = Google
* AS3 = a Japanese university
* AS4 = someone running a VPS in Phoenix

All of them **need to send data to each other** - fast, cheap, reliable.

Instead of:

> "Send your data across oceans, pay a transit fee, and hope it gets there…"

They say:

> "Let's all plug into the **same building**, connect directly, and swap packets *locally.*"

This **building full of connections** is called an **IXP**.

| Real-World Equivalent          | IXP Version              |
| ------------------------------ | ------------------------ |
| Post office                    | Router/switch            |
| Road or train track            | Ethernet or fiber cables |
| City district for post offices | Data center/IXP campus   |

Without IXPs:

* Traffic would take **longer routes**
* Be **more expensive** (paying third-party transit)
* **Slower**, more congested

With IXPs:

* **Faster** access to services (Netflix, YouTube, email...)
* **Reduced latency**
* **Direct peering** = less middleman

---

## TCP - head-of-line blocking

**TCP is all about reliability AND order.** And in TCP's world, **order matters as much as content**. It wants no loss, no duplicates and no reordering.

Say you send 4 packets (segments):

```txt
Segment 1 ✅
Segment 2 ❌ (gets lost)
Segment 3 ✅
Segment 4 ✅
```

TCP Receiver would say:

> "Where's Segment 2? I'm not touching 3 or 4 until 2 arrives. I want the full story, *in order.*"

This is called **head-of-line blocking.**

TCP block because it **buffers everything** until it can give it to the application in **perfect order**. It doesn't want to **risk confusion or partial delivery.**

So if segment 2 is missing, TCP would keep 3 and 4 in a waiting room. It asks the sender: "Hey, resend 2 please?" Only once 2 comes back, it gives all 1-4 to the app.

---

> Even if the receiving application reorders everything in the end, **TCP still insists on preserving the order during transmission.**

| Layer         | Responsibility             | Example                            |
| ------------- | -------------------------- | ---------------------------------- |
| TCP (Transport Layer 4) | Guarantee order + delivery | Keeps segments 1,2,3,4 in order    |
| App (Application Layer 7) | Use the final data only    | Renders a video, opens a file, etc |

Let's imagine TCP **didn't** care about order and gave your app this mess:

```
Segment 1 ✅
Segment 3 ✅
Segment 4 ✅
Segment 2 ❌ (still waiting)
```

Now your **application layer** would have to:

* Detect that 2 is missing
* Figure out how long to wait
* Deal with out-of-order data
* Possibly get stuck midway

That's **super error-prone** and way too stressful for your poor app.

So with TCP, your app doesn't have to worry about retransmissions. And you don't need to design reordering logic. It works the same on any OS, browser, or network

---

### UDP

Why not just send everything and fix later? That's exactly what **UDP** does.

UDP is like:

> "Just send it! YOLO! If something drops, *meh!* You figure it out."

Perfect for:

* Video calls
* Games
* Live streaming

But not for:

* Web pages
* File downloads
* Secure communication (SSL/TLS)

---

## DNS & censorship

> **Using HTTPS + DoH helps fight censorship, but only if we don't centralize all DNS power into the hands of one big company.**

Most traffic today is encrypted over HTTPS. It **hides the content** of what you're loading (e.g., images, HTML). It **protects privacy** by encrypting requests/responses. And censors can't just peek inside or tamper easily. So blocking specific **content** becomes harder.

Even if your web traffic is encrypted (HTTPS), your **DNS requests** - like:

```
Where is google.com?  
Where is nytimes.com?  
```

... can still **be intercepted** if you use **normal DNS (port 53)**, which is plaintext.

**That's where DoH (DNS over HTTPS) comes in**. It encrypts DNS queries **inside HTTPS**, so the censor **can't see or block what site you're trying to access**, even at the DNS level. Then, DNS is hidden. The server you're talking to looks just like normal HTTPS. You blend in with the crowd.

**DoH makes censorship harder**, but **if all DoH resolvers are centralized (e.g., only Cloudflare or Google)**, then *they* can become censors too. Like if everyone uses Google's DoH, but gov tells Google to block some domains, then **centralized censorship**, just through a "trusted" DoH provider.

What's the ideal? A **decentralized network** of DoH resolvers, some self-hosted, some community-run. So no single actor controls access.

Like, we don't want a world where your encrypted DNS still has a backdoor held by some megacorp. HTTPS hides the *what*, DoH hides the *where*, but **freedom** depends on *who* answers your questions.

---

## HTTPS & MitM

> **HTTPS encrypts the contents** of your request:

* The **URL path**, like `/search?q=xx`
* Your **cookies**, **headers**, **data**

> But it does **NOT hide**:

* The **IP address** you're talking to
* Your own IP address

This is because IP addresses live in the **IP header**, and that layer is *outside* the HTTPS encryption. HTTPS runs at the **Application Layer**, and IP lives way down in the **Network Layer**.

But that **doesn't** mean a **man-in-the-middle** (MitM) could modify the origin and destination IP addresses. **Only if in very limited, local, or malicious router conditions.**

Normally, MitM **can't modify** the IPs in-transit.

Because:

1. **Routers** don't rewrite source/destination IPs unless NAT is involved
2. Every packet has to know **where to go**
3. If someone **modifies the IPs**, the packets either:

   * Get dropped,
   * Go to the wrong place,
   * Or cause weird connection failures

Only in these scenarios **can** a MitM mess with IPs:

| Scenario                                      | Can MitM change IPs? | Why                                         |
| --------------------------------------------- | -------------------- | ------------------------------------------- |
| **Normal internet**                           | No                 | Would break the routing                     |
| **Local network hijack** (e.g., rogue router) | Yes                | Can spoof your gateway                      |
| **Malware on device**                         | Yes                | Has full control of your traffic            |
| **Using NAT** (e.g., in your router)          | But expected       | NAT rewrites IPs legitimately               |
| **Government firewall**           | Yes                | Acts as an active MITM or transparent proxy |

Even if they **can't decrypt your HTTPS** or rewrite IPs, they can still:

* **See your IP and the server's IP**
* Know that "you talked to 142.250.72.206" (which is Google)
* Do **traffic analysis** (timing, size, frequency)
* Use **DNS leaks** or SNI (unless encrypted with ESNI/ECH) to guess what you're accessing

So IPs leak *who talks to whom*, even if not *what is said*. That's why tools like **Tor** or **VPNs** try to hide your origin IP too.