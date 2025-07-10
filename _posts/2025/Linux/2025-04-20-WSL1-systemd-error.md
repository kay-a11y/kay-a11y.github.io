---
layout: post
title: "[Fix] WSL1: systemd upgrade error - `E: Sub-process /usr/bin/dpkg returned an error code (1)`"
description: Fixing the infamous dpkg/systemd error on WSL1 by upgrading to WSL2 with full compatibility.
date: 2025-04-20 17:17:00 +0800
categories: [🤖 tech, 🐧 Linux]
tags: [🐧 Ubuntu, 🧡 Ubuntu, ⚙️ WSL2, 💣 systemd error, 🧠 BIOS virtualization, 🛠️ troubleshooting, 🪟 Windows11, 👩🏻‍💻 glow-up]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/WSL_Ubuntu.jpg
---

## 🔥 THE PROBLEM

On **WSL (Windows Subsystem for Linux) WSL1**, you ran:

```bash
sudo apt update && sudo apt upgrade -y
```

But then it slapped you with this nasty surprise:

```
Failed to take /etc/passwd lock: Invalid argument
dpkg: error processing package systemd (--configure):
installed systemd package post-installation script subprocess returned error exit status 1
```

And that very last line:

```
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

## 💡 THE MOST RELIABLE FIX:

That's because WSL1 **doesn't support** systemd at all-it's like trying to install a kitchen sink in a cardboard box. 💀

### ✅ Step 1: Confirm if you're in WSL1 or WSL2

```bash
wsl.exe -l -v
```

You'll see a list like:

```
  NAME      STATE           VERSION
* Ubuntu    Running         1
```

> If VERSION is `1` → **WSL1**.  
> If VERSION is `2` → 🎉 You good! (though the error suggests you're still on 1.)

---

### 🧨 Step 2: If you're on WSL1 - **Upgrade to WSL2**

Because WSL2 **supports systemd natively** now. Here's how:

1. **Ensure you're on Windows 10 (Build 19041+) or Windows 11**
2. Run in PowerShell (as admin):

```powershell
wsl --set-version Ubuntu 2
```

3. Set WSL2 as default:

```powershell
wsl --set-default-version 2
```

That command will switch your WSL into **2025-worthy mode**-with full systemd compatibility and no more /etc/passwd lock failures.

---

## 💥 If then You're Hitting the Classic `ERROR_NOT_SUPPORTED`

That line:

```
Error code: Wsl/Service/CreateVm/HCS/ERROR_NOT_SUPPORTED
```

translates to:  
> "Yo dude, your system can't even support WSL2 VM tech. Sorry not sorry."

This isn't just a config issue anymore-it means **your system doesn't meet WSL2's requirements**.

---

We need to **check 3 things one by one** to make sure WSL2 *can even* work on your current setup:

### 🧩 1. Is Virtualization Enabled in BIOS?

- Reboot into BIOS/UEFI (usually by pressing `Delete` or `F2` when booting).
- Look for **Intel VT-x**, **AMD-V**, or anything saying **Virtualization** - and enable it.
- Save and reboot.

#### Detailed Guide - Enable Virtualization in BIOS

**e.g. On GALAX B560 LIGHTXY motherboard:**

##### 🛠️ Steps to Enable Virtualization (Intel VT-x):

1. **Enter BIOS Setup**:
   - Restart your computer.
   - Immediately press the `Delete` key repeatedly as your system boots up to enter the BIOS setup utility.

2. **Navigate to Advanced Settings**:
   - Use the arrow keys to move to the **Advanced** tab in the BIOS menu.

3. **Locate CPU Configuration**:
   - Within the Advanced tab, find and select **CPU Configuration**.

4. **Enable Intel Virtualization Technology**:
   - Look for an option named **Intel Virtualization Technology** or **VT-x**.
   - Select this option and change its setting to **Enabled**.

5. **Save and Exit**:
   - Press `F10` to save your changes and exit the BIOS.
   - Confirm any prompts to apply the changes.

### 🧩 2. Is Hyper-V Platform Installed?

Even if WSL2 is "set-default," it **needs Hyper-V tech** to *create a virtual machine backend*. So run this in **PowerShell (Admin)**:

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

Optional but helpful (for GUI apps later):

```powershell
dism.exe /online /enable-feature /featurename:HypervisorPlatform /all /norestart
```

### 🧩 3. Check Windows Version - WSL2 Minimum = Win10 2004+

Run this in PowerShell:

```powershell
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
```

If you see something like:

```
OS Version:                10.0.18362  <== ❌ too low (must be ≥ 19041)
```

Then, that's the real block. You'll need to **update to at least Windows 10 2004 or later**, or ideally **Windows 11**.

### 🧩 4. Reboot‼️

Once all three conditions are met, **reboot your PC** to let Windows fully apply the updated settings.

---

### 🐾 TL;DR

| Thing                  | Status                    | Fix                                                                 |
|------------------------|---------------------------|----------------------------------------------------------------------|
| BIOS Virtualization    | ❓ Maybe Disabled          | Enable it in BIOS                                                   |
| Hyper-V/VM Support     | ❌ Missing                 | Run `dism` commands to enable features                              |
| OS Version Too Low     | ❓ Check with `systeminfo` | Must be Win10 2004+ or upgrade to Windows 11                        |
| Can't Use WSL2 Now     | ✅ True                    | Use script workaround + avoid `systemd` packages on WSL1            |

---

## Solved

Run:

```bash
sudo apt update && sudo apt upgrade -y
```

inside your Ubuntu shell, it should work.


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
