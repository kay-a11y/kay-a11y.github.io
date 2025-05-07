---
layout: post
title: "📝 How I Successfully Fixed the Dual Boot Menu (Pop!\_OS + Windows)"
description: Successfully fixed the missing Windows boot option in systemd-boot with Pop!_OS 22.04. Step-by-step guide covering ESP partition issues, windows.conf mistakes, and the final working solution.
date: 2025-05-08 03:41:00 +0800
categories: [🤖 tech, 🐧 Linux]
tags: [🐧 Pop!_OS, 🪟 Windows11, 🖥 dual boot, 🧠 systemd-boot, 💾 ESP partition, 🐞 Linux troubleshooting]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/popos.png
---

## 🖥️ **The Problem**

Even after having both **Pop!\_OS** and **Windows 11** installed, only Pop!\_OS and "Reboot Into Firmware Interface" were showing in the systemd-boot menu. Windows wouldn’t appear as a boot option.

---

## 🔧 **Step-by-Step Solution**

### 1️⃣ **Check if the Windows Boot Loader Exists**

```bash
sudo mkdir /mnt/nvmeboot
sudo mount /dev/nvme0n1p1 /mnt/nvmeboot
ls /mnt/nvmeboot/EFI/Microsoft/Boot/bootmgfw.efi
```

✅ If the file `/mnt/nvmeboot/EFI/Microsoft/Boot/bootmgfw.efi` exists, the Windows bootloader is present.

---

### 2️⃣ **Copy the Microsoft Boot Folder to Pop!\_OS ESP**

```bash
sudo cp -r /mnt/nvmeboot/EFI/Microsoft /boot/efi/EFI/
```

This puts the Windows bootloader into Pop’s EFI partition so **systemd-boot** can find it.

---

### 3️⃣ **Fix the Loader Entry for Windows**

First, open the Windows loader entry:

```bash
sudo nano /boot/efi/loader/entries/windows.conf
```

It might look like this:

```bash
title   Windows 11
efi     /EFI/Microsoft/Boot/bootmgfw.efi
options rootfstype=ntfs root=UUID=xxxx
```

👉 **Important:** Delete the line starting with `options`:

```bash
options rootfstype=ntfs root=UUID=xxxx
```

Why? → **systemd-boot doesn’t need (and may even break with) the `options` line for Windows.**
Windows handles its own options internally.

So the final `windows.conf` should look like:

```bash
title   Windows 11
efi     /EFI/Microsoft/Boot/bootmgfw.efi
```

---

### 4️⃣ **Update systemd-boot Entries**

```bash
sudo update-initramfs -c -k all
sudo bootctl update
```

Optional but recommended:

```bash
sudo bootctl install
```

---

### 5️⃣ **Check Boot Loader Status**

```bash
sudo bootctl status
```

Make sure it shows both Pop!\_OS and Windows 11 in the boot entries.

Or list the loader entries directly:

```bash
sudo bootctl list
```

---

## 🏁 **Final Result**

After rebooting, the systemd-boot menu correctly displayed:

```
Pop!_OS
Windows 11
Windows Boot Manager
Reboot Into Firmware Interface
```

✅ **Both OS choices work perfectly. Mission accomplished.**

---

### 💡 **Key Lessons Learned**

* Pop!\_OS uses **systemd-boot**, not GRUB.
* Windows loader must be copied to the **same ESP** Pop!\_OS uses.
* The `options` line in `windows.conf` should be removed for systemd-boot compatibility.
* BIOS boot priority should point to **Linux Boot Manager** or the Pop!\_OS drive first.
* `bootctl update` and `bootctl install` are essential after changing boot entries.

---

## ⚠ What if Windows Updates Break the Boot Entry?

Since the Microsoft bootloader (`bootmgfw.efi`) was **copied** (not symlinked) into Pop!_OS's ESP (`/boot/efi/EFI/Microsoft/`), there’s a possibility that:

- Major Windows updates (especially feature updates) might **replace or update** the bootloader file **only on the original Windows ESP (nvme0n1p1)**.
- The copied version in the Pop!_OS ESP would then become outdated or invalid.

---

### 🔄 How to Fix It (If Windows Stops Booting from systemd-boot Menu):

1️⃣ Boot into Pop!_OS.  
2️⃣ Mount the Windows EFI partition again:

```bash
sudo mkdir -p /mnt/windows-esp
sudo mount /dev/nvme0n1p1 /mnt/windows-esp
```

3️⃣ Re-copy the updated bootloader:

```bash
sudo cp -r /mnt/windows-esp/EFI/Microsoft/Boot /boot/efi/EFI/Microsoft/
```

4️⃣ Verify the file exists:

```bash
ls /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi
```

5️⃣ Reboot. The Windows entry should work again.

---

## 💡 Pro Tip:

This method is robust for most cases. But if you often get large Windows updates or upgrade to new major releases (like from Windows 11 23H2 to 24H2), remember to repeat the copy step.

If you plan to **remove Windows completely later**, this won’t matter because the copied bootloader will no longer be needed. 🎉

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
