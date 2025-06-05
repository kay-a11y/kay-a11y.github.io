---
layout: post
title: "Crack That ZIP - Using John + Hashcat (`No hashes loaded` Error Fix Included)"
description: "Walkthrough for cracking encrypted ZIP files using zip2john + Hashcat. Includes how to solve the dreaded 'No hashes loaded' error when dealing with multi-file archives. Battle-tested and beginner-friendly."
date: 2025-06-05 07:25:00 +0800
categories: [🤖 tech, 🔒 Web Security]
tags: [🐧 Linux, 🐾 Penetration, 🐾 Hacker Basics, 🔒 Web Security, 🔒 Privacy, 😼 Hashcat, 🧔 JohnTheRipper, 🧪 zip2john, 🔒 Password Cracking, 🖥️ CLI, 🐾 Brute Force]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: 
---

> **Disclaimer:**  
> This post is shared **strictly for educational and ethical purposes**.  

> 🧪 Missed Part 1?  
> 👉🏻 Dive into [Hashcat Basics](https://kay-a11y.github.io/posts/hashcat-basics/) first - that's where the magic begins. 

---

## 🧨 Install `zip2john` (John Jumbo Build)

### 🛠 1. Install build dependencies

```bash
sudo apt update
sudo apt install -y build-essential git zlib1g-dev libssl-dev libbz2-dev libgmp-dev libpcap-dev pkg-config
```

---

### 💻 2. Clone the Jumbo repo

```bash
git clone https://github.com/openwall/john.git
cd john/src
```

---

### 🪛 3. Build it

```bash
./configure && make -s clean && make -sj$(nproc)
```

* This might take a minute or two.
* It will compile the full John suite, including `zip2john`, `rar2john`, `pdf2john`, and all the goodies

---

### ✅ 4. Verify it's installed

```bash
cd ../run
./zip2john
```

If that works, you're golden.

---

### 🐾 Optional: Add it to your `$PATH`

So you can use it from anywhere:

```bash
echo 'export PATH="$HOME/john/run:$PATH"' >> ~/.zshrc 
# echo 'export PATH="$HOME/john/run:$PATH"' >> ~/.bashrc 
source ~/.zshrc 
# source ~/.bashrc 
```

or:

```bash
echo 'export PATH="$HOME/john/run:$PATH"' >> ~/.bashrc 
source ~/.bashrc 
```

Now you can just do:

```bash
zip2john your.zip
```

From any folder.

---

## 🧪 Test the Encrypted ZIP

I messed around with ZIP files *a lot* during this part.

If the archive contains **multiple files**, but I keep using `17200` (the **single-file Hash-Mode**), then after `john` dumps the hash, **Hashcat** will always throw an error:

> `No hashes loaded.`

---

### 🔨 0. Solve the error `No hashes loaded` in Hashcat

Here are the `example_hashes` from [hashcat.net/wiki](https://hashcat.net/wiki/doku.php?id=example_hashes){:target="_blank"}.

```txt
Hash-Mode 	Hash-Name 	Example 
17200 	PKZIP (Compressed) 	$pkzip2$1*1*2*0*e3*1c5*eda7a8de*0*28*8*e3*eda7*5096*a9fc1f4e951c8fb3031a6f903e5f4e3211c8fdc4671547bf77f6f682afbfcc7475d83898985621a7af9bccd1349d1976500a68c48f630b7f22d7a0955524d768e34868880461335417ddd149c65a917c0eb0a4bf7224e24a1e04cf4ace5eef52205f4452e66ded937db9545f843a68b1e84a2e933cc05fb36d3db90e6c5faf1bee2249fdd06a7307849902a8bb24ec7e8a0886a4544ca47979a9dfeefe034bdfc5bd593904cfe9a5309dd199d337d3183f307c2cb39622549a5b9b8b485b7949a4803f63f67ca427a0640ad3793a519b2476c52198488e3e2e04cac202d624fb7d13c2*$/pkzip2$
17210 	PKZIP (Uncompressed) 	$pkzip2$1*1*2*0*1d1*1c5*eda7a8de*0*28*0*1d1*eda7*5096*1dea673da43d9fc7e2be1a1f4f664269fceb6cb88723a97408ae1fe07f774d31d1442ea8485081e63f919851ca0b7588d5e3442317fff19fe547a4ef97492ed75417c427eea3c4e146e16c100a2f8b6abd7e5988dc967e5a0e51f641401605d673630ea52ebb04da4b388489901656532c9aa474ca090dbac7cf8a21428d57b42a71da5f3d83fed927361e5d385ca8e480a6d42dea5b4bf497d3a24e79fc7be37c8d1721238cbe9e1ea3ae1eb91fc02aabdf33070d718d5105b70b3d7f3d2c28b3edd822e89a5abc0c8fee117c7fbfbfd4b4c8e130977b75cb0b1da080bfe1c0859e6483c42f459c8069d45a76220e046e6c2a2417392fd87e4aa4a2559eaab3baf78a77a1b94d8c8af16a977b4bb45e3da211838ad044f209428dba82666bf3d54d4eed82c64a9b3444a44746b9e398d0516a2596d84243b4a1d7e87d9843f38e45b6be67fd980107f3ad7b8453d87300e6c51ac9f5e3f6c3b702654440c543b1d808b62f7a313a83b31a6faaeedc2620de7057cd0df80f70346fe2d4dccc318f0b5ed128bcf0643e63d754bb05f53afb2b0fa90b34b538b2ad3648209dff587df4fa18698e4fa6d858ad44aa55d2bba3b08dfdedd3e28b8b7caf394d5d9d95e452c2ab1c836b9d74538c2f0d24b9b577*$/pkzip2$
17220 	PKZIP (Compressed Multi-File) 	$pkzip2$3*1*1*0*8*24*a425*8827*d1730095cd829e245df04ebba6c52c0573d49d3bbeab6cb385b7fa8a28dcccd3098bfdd7*1*0*8*24*2a74*882a*51281ac874a60baedc375ca645888d29780e20d4076edd1e7154a99bde982152a736311f*2*0*e3*1c5*eda7a8de*0*29*8*e3*eda7*5096*1455781b59707f5151139e018bdcfeebfc89bc37e372883a7ec0670a5eafc622feb338f9b021b6601a674094898a91beac70e41e675f77702834ca6156111a1bf7361bc9f3715d77dfcdd626634c68354c6f2e5e0a7b1e1ce84a44e632d0f6e36019feeab92fb7eac9dda8df436e287aafece95d042059a1b27d533c5eab62c1c559af220dc432f2eb1a38a70f29e8f3cb5a207704274d1e305d7402180fd47e026522792f5113c52a116d5bb25b67074ffd6f4926b221555234aabddc69775335d592d5c7d22462b75de1259e8342a9ba71cb06223d13c7f51f13be2ad76352c3b8ed*$/pkzip2$
17225 	PKZIP (Mixed Multi-File) 	$pkzip2$3*1*1*0*0*24*3e2c*3ef8*0619e9d17ff3f994065b99b1fa8aef41c056edf9fa4540919c109742dcb32f797fc90ce0*1*0*8*24*431a*3f26*18e2461c0dbad89bd9cc763067a020c89b5e16195b1ac5fa7fb13bd246d000b6833a2988*2*0*23*17*1e3c1a16*2e4*2f*0*23*1e3c*3f2d*54ea4dbc711026561485bbd191bf300ae24fa0997f3779b688cdad323985f8d3bb8b0c*$/pkzip2$
17230 	PKZIP (Mixed Multi-File Checksum-Only) 	$pkzip2$8*1*1*0*8*24*a425*8827*3bd479d541019c2f32395046b8fbca7e1dca218b9b5414975be49942c3536298e9cc939e*1*0*8*24*2a74*882a*537af57c30fd9fd4b3eefa9ce55b6bff3bbfada237a7c1dace8ebf3bb0de107426211da3*1*0*8*24*2a74*882a*5f406b4858d3489fd4a6a6788798ac9b924b5d0ca8b8e5a6371739c9edcfd28c82f75316*1*0*8*24*2a74*882a*1843aca546b2ea68bd844d1e99d4f74d86417248eb48dd5e956270e42a331c18ea13f5ed*1*0*8*24*2a74*882a*aca3d16543bbfb2e5d2659f63802e0fa5b33e0a1f8ae47334019b4f0b6045d3d8eda3af1*1*0*8*24*2a74*882a*fbe0efc9e10ae1fc9b169bd060470bf3e39f09f8d83bebecd5216de02b81e35fe7e7b2f2*1*0*8*24*2a74*882a*537886dbabffbb7cac77deb01dc84760894524e6966183b4478a4ef56f0c657375a235a1*1*0*8*24*eda7*5096*40eb30ef1ddd9b77b894ed46abf199b480f1e5614fde510855f92ae7b8026a11f80e4d5f*$/pkzip2$
```

The examples below assume the ZIP file is **Compressed** and contains **only one file**, which means you should use **Hash-Mode** `17200`.

---

### ✅ 1. Extract hash from Encryted zip

```bash
zip2john -s "~/test.zip" > /tmp/test_zip.john
```

This raw `.john` would be like:

```txt
test.zip:$pkzip$8*1*1*0*8*24*d9...ed69b2b*$/pkzip$::test.zip:test/myfile.txt:/home/username/test.zip
```

---

#### 🥡 About `/tmp/`

You *can*:

* Read/write/execute files from `/tmp/` in **any program or path**.
  Example:

  ```bash
  hashcat -m 17200 /tmp/test_zip.john rockyou.txt
  ```

You *cannot*:

* **Count on them being permanent.**

  * Most Linux distros **auto-clean `/tmp/` on reboot** or periodically.

If it's a file you **don't wanna lose**, move or copy it somewhere:

```bash
mv /tmp/multi_zip.john ~/Documents/hashcat/zip_hash_clean.txt
```

Then crack that sucker from the new path.

---

### ✅ 2. Strip the `zip2john` label and save as separate file

This created the file in your current directory:

```bash
cut -d ':' -f2 /tmp/test_zip.john > zip_hash_clean.txt
```

This clean `.txt` would be like:

```txt
$pkzip$8*1*1*0*8*24*d9...ed69b2b*$/pkzip$
```

#### ✅ 3. Run Hashcat

```bash
hashcat -m 17200 -a 3 zip_hash_clean.txt '?a?a?a?a?'
```

---

## 🎭 Some Doable Mask Ideas

Try testing with these masks - let's see where your password starts to stumble.

* `rockyou.txt`
* `rockyou.txt -r rules/best64.rule`

When `rockyou.txt` fails and you're outta patience?  
Bring in the masks.

* `?d?d?d?d` - 4-digit PINs (phone locks, dumb zips, grandpa's secrets)
* `?d?d?d?d?d?d?d?d` - 8-digit PINs (lazy but common)
* `?l?l?l?l?l?l` - lowercase words (`secret`, `monkey`, `hunter`)
* `?u?l?l?l?l?l` - capital + lowercase (`Admin1`, `Jesus7`)
* `?l?l?l?l?d?d` - classic endings (`music88`, `cool99`)
* `?l?l?d?d?l?l` - creative combos (`co88de`, `py33rs`)
* `?a?a?a?a` - full wildcard madness when you've lost all hope

💡 Use `--increment` to let Hashcat automatically test shorter → longer combos.  
Save your fingers, save your time:

* `'?a?a?a?a?a?a?' --increment --increment-min 4 --increment-max 6`
* `'?d?d?d?d?d?d?d?d' --increment --increment-min 7 --increment-max 8`
* `'?l?l?l?l?l?l?l?l' --increment --increment-min 7 --increment-max 8`

---

### ✍🏻️ Customize Your `guesslist.txt`

If you've exhausted them all, consider using `itertools` to customize your own `guesslist.txt`.

```py
import itertools

# Define your keyword list
keywords = ['2025', '0605', 'somenames']
# Generate combinations of 1 and 3 parts
with open("/PATH/TO/YOUR/guesslist.txt", "w") as f:
    for r in range(1, 4):
        for combo in itertools.product(keywords, repeat=r):
            f.write(''.join(combo) + '\n')
```

Then:

```bash
hashcat -m 17200 zip_hash_clean.txt guesslist.txt
```

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
