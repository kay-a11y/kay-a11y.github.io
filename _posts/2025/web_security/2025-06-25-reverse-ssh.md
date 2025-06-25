---
layout: post
title: "Bypassing CGNAT with Reverse SSH"
description: 
date: 2025-06-25 01:39:00 +0800
categories: [🤖 tech, 🔒 Web Security]
tags: [🐧 Linux, 💻 Networking, 🛡️ CGNAT, 🚇 SSH, 🌍 AWS, 🏠 HomeLab, 🤖 Cloud, 🌐 Remote-Access]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: 
---

## What is Reverse SSH

It's like instead of *you* SSH'ing into your home PC, your **home PC SSHs out** to a VPS you own. Then later, *you use another device ssh to that VPS*, and secretly slip back into your home PC - through that tunnel.

It **reverses the connection direction**, so even if you're behind **CGNAT**, **firewalls**, or **double NAT**, you can still reach your machine from the outside.

In **CGNAT**, your home PC can go **outbound**, but can't accept **inbound connections** (like normal SSH). So **direct `ssh` to your home (Public) IP** is blocked.

> Also you can check this post -> [here](https://kay-a11y.github.io/posts/how-internet-works/#ipv4--shared-ips){:target="_blank"}

---

## How to Reverse SSH

Here's how to use an AWS EC2 instance + a sidekick device (like a Raspberry Pi) to punch through CGNAT and tunnel your way back into a hidden PC using reverse SSH.

### Prep - Key

Send your private key from your PC to your Raspberry Pi (over SSH)

**Suppose:**

* Your PC has the key at: `/home/you/key.pem`
* Your Pi username is: `pi`
* Your Pi's IP is: `192.168.1.77` (replace with your actual Pi IP)

**A. Use `scp` (secure copy):**

On your **PC**:

```bash
scp /home/you/key.pem pi@192.168.1.77:/home/pi/
```

* Enter your **Pi password** when prompted.
* The key will be copied to `/home/pi/key.pem` on your Pi.

**B. Set proper permissions on your Pi (important):**

SSH into your Pi:

```bash
ssh pi@192.168.1.77
```

Then, on your Pi:

```bash
chmod 600 ~/key.pem
```

---

### Security Group

check your AWS EC2 security group and make sure port `29999` (or any port you choose for reverse SSH) is open to the outside world, so you (or anyone you allow) can connect *back* through that tunnel.

**From the AWS Console**:

1. Log in to [AWS Console](https://console.aws.amazon.com/ec2).
2. Go to **EC2 > Instances**, select your instance.
3. In the "Description" panel, find **Security Groups**. Click the group name.
4. In the security group details, look under **Inbound rules**.
You want to see a rule like this, or add this rule if not:

```txt
Type: Custom TCP Rule
Port Range: 29999
Source: 0.0.0.0/0  (or your public IP for safety)
```

---

### Check and update EC2's `sshd_config`

1. SSH into your EC2.

2. Edit the config file:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

3. **Make sure these lines are present and set like this:**

   ```
   AllowTcpForwarding yes
   GatewayPorts yes
   ```

4. Save and restart SSH:

   ```bash
   sudo systemctl restart ssh
   ```

---

### Test the Tunnel

**On your PC (at home, behind CGNAT):**

```bash
ssh -i "/your/ec2/key.pem" \
    -R 29999:localhost:22 \
    ubuntu@ec2-12-123-234-56.ap-northeast-1.compute.amazonaws.com
```

*(Keep this session open!)*

* `-R 29999:localhost:22` means:

  > Listen on port 29999 of the VPS, and forward everything back to port 22 (SSH) of my home PC.

After you run that, as long as the SSH session is active, **then from anywhere (or from your other device):**

SSH to your PC **via the VPS**:

```bash
ssh -i "/your/ec2/key.pem" \
    -p 29999 \
    pc_username@ec2-12-123-234-56.ap-northeast-1.compute.amazonaws.com
```

* This means "Connect to the VPS at port 29999, which is now tunneling to your home PC's SSH."

Type your PC password and then voila!

---

### Trouble Shooting: Make sure your home PC allows password login

* Check `/etc/ssh/sshd_config` for `PasswordAuthentication yes`
* Restart sshd if you change it (`sudo systemctl restart ssh`)

---

## To PROVE You've Arrived Home

```bash
# 1. Check the hostname
hostname

# 2. Check your internal IP
# You'll see `192.168.x.x` or `10.x.x.x`- your home network, 
# not EC2's `172.31.x.x`.
ip a 

# 3. Check your OS
lsb_release -a

# or
cat /etc/os-release

# 4. List your home files

ls ~

# 5. Print your home username
whoami
```