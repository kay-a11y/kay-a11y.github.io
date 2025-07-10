---
layout: post
title: "How Curiosity Hijacked Me: Cracking My Router's Hidden Token"
description: "A Full Reverse Engineering Walkthrough of Dynamic Login Token and SHA-256 Authentication"
date: 2025-04-28 01:26:00 +0800
categories: [🤖 tech, 🔒 Web Security]
tags: [🧪 Reverse Engineering, 📜 JavaScript, 🛜 Router Hacking, 🔒 Web Security, 💻 Token-Based Authentication, 🐾 Personal Journey]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/printMeow.png
---

## 🌸 Part 1 - Opening and Story Start

So originally, I was planning to sit down quietly and study TCP/IP protocol theory seriously.  

![networkBasics](/assets/img/posts/networkBasics.jpg)

But then... curiosity dragged me away.

I tried to find the WAN and WLAN Ports on my router.  
This *tiny hands-on move* totally distracted me from the boring theory and pulled me into something dirty and wild. 🐾  
One second I was a diligent student, the next I was hacking like a cyberpunk gremlin. 🛜✨

Before we dive deep, let's first figure out what these words **WAN**, **LAN**, and **WLAN** actually mean, and know a bit about **Do Routers Use HTTP for Login**.

---

### 😇 What are WAN, LAN, and WLAN?

Imagine your **router** at home like a tiny, magical **post office**.  
It connects *you* (and all your devices) to the *whole big Internet world* 🌍.

Inside and around this magical post office:

#### ✨ WAN - Wide Area Network

- **WAN** is *the giant highway* 🚚 outside your house (your router).
- It connects your home to the **Internet** itself.
- **WAN Port on Router?** YES. That's usually where your Internet Service Provider (ISP) cable plugs in!

👉 Without WAN, your house would be totally cut off - no Netflix, no hacking tutorials, no digital adventures. 😭

---

#### ✨ LAN - Local Area Network

- **LAN** is everything *inside your house*. 🏡
- It's like private roads connecting your devices - your PC, your laptop, your tablet, your fridge even (if it's smart enough).
- **LAN Ports on Router?** YES - those ports labeled **LAN1**, **LAN2**, etc.

👉 LAN is like family members talking happily inside the house, no need to go out onto the highway.

---

#### ✨ WLAN - Wireless Local Area Network

- **WLAN** is *LAN without cables* - wireless 🛜✨.
- All your WiFi devices connect to the router using invisible radio waves.

👉 WLAN is like your family members shouting happily across rooms using walkie-talkies instead of cables. 📣📡

---

#### Quick Recap 📦

| Term | What It Is | Where You Find It |
|:---|:---|:---|
| WAN | Your router's connection to the **Internet** | WAN port (often blue) |
| LAN | **Wired** connections inside your home | LAN1, LAN2 ports |
| WLAN | **Wireless** connections inside your home | WiFi broadcast |

---

### 🛜 In General: Do Routers Use HTTP for Login?

#### 1. **Home Routers** (like TP-Link, Netgear, D-Link...)

✅ **Most default to HTTP**, because:

- Easier for grandma to connect and not get SSL warnings 😂
- Certificates (for HTTPS) are messy to generate and maintain
- ISPs (especially in China) don't want to pay for SSL certs for *each* router they ship
- Local network assumed to be "safe" (*cough cough bullshit*)  
- They just hash or lightly obfuscate the password inside the page

**Example:**  
- `http://192.168.1.1`
- `http://192.168.0.1`
- `http://192.168.2.1`

---

#### 2. **Business/Enterprise Routers** (Cisco, Ubiquiti, Fortinet...)

✅ **Professional routers often enforce HTTPS** by default.  
✅ Their login page is something like:

- `https://192.168.1.1`
- or they force you to upload your own SSL cert.

They know that even inside a corporate LAN, attackers can sniff.  
**HTTPS everywhere** becomes essential.

---

#### 3. **Modern Premium Consumer Routers** (Asus, Synology, Unifi...)

✅ Newer models **force HTTPS** too (even self-signed).

When you access them:

- Your browser will give you a "Warning: Not Trusted" page (because SSL cert is self-signed)
- You have to click "Proceed anyway" to enter.

But at least the channel is **fully encrypted** 🔒!

---

#### 🖤 So the ugly truth is:

| Router Type | Default Login Mode | Notes |
|:---|:---|:---|
| Cheap ISP home routers | HTTP | Maybe password hashed, but sniffable |
| Enterprise routers | HTTPS | Strong encryption, professional-grade |
| New expensive consumer routers | HTTPS (self-signed) | Still better, but warnings appear |

---

## Part 2 - Discovery: From Bored to Completely Obsessed

After understanding WAN, LAN, and WLAN basics,  
I decided to **log into my router** to explore more settings.

---

### 🛜 How I Accessed My Router

1. I connected to the router (via WiFi).
2. Opened my browser.
3. Typed `http://192.168.2.1/` into the address bar.

🎯 Boom - a login page popped up, asking for:

- **Username** (mine was `admin`)
- **Password** (on my sticker, it was `admin`)

---

### 🍬 A Little Unexpected Curiosity...

Once I logged in, I opened the browser DevTools (`F12`) to snoop around...  
And that's where I noticed something **strange**:

In the **Network** tab, when the login request (`POST`) was sent,  
the "Password" field **wasn't my plain password**.

Instead, it was a weird, long string like:

```
5fd0828219e305d95e21e4925c42faa32970e3702bcf4a23863133e5f7574b39
```

👀 My password `admin` obviously didn't look like *that*.  
**Something** was happening behind the scenes!

---

### 🌸 First Guess: Maybe It's Hashed?

I guessed:

- Maybe the browser hashed the password before sending it?  
- Maybe it used **SHA-256**, **MD5**, or even **HMAC**?

So, I tested immediately:
1. Went to an online SHA-256 tool.
2. Hashed `admin`.
3. Got:
   ```
   6f021f1c9d1cff7fc1f9fa533025e47ee6b2360bdde03c865e11b4c60f9982c8
   ```

🚨 But it **still didn't match** the login POST password!

Something trickier was happening...  
and that made my curiosity catch fire even harder. 

---

## Part 3 - Exploring JavaScript: Hunting the Encryption

After realizing my password was somehow being **transformed** before it was sent...  
I knew it was time to dig deeper. 🥷

Where do browsers hide their dirty tricks?  
👉 In **JavaScript**.

---

### 🛜 My Plan: Tear Open the Source Code

1. I stayed on the router login page.
2. Opened **DevTools** → switched to the **Sources** tab.
3. Hit `Ctrl+Shift+F` to **search across all scripts**.

I searched for keywords like:

- `password`
- `hash`
- `encrypt`
- `login`

I knew somewhere, buried in the scripts,  
there would be a **function** that touched my password before sending it.

---

### 🍬 What I Found: The Critical Function

Hidden inside the main `index` JavaScript file,  
![](/assets/img/posts/post_filename.jpg)  
I found this function:

```javascript
function g_loginToken(xml)
{
  var xmlObj = $(xml).text();
  var Password = "";
  var IsAutoShowPass = $("#IsAutoShowPass").val();
  var frm_Password = $("#Frm_Password").val();
  if (1 == IsAutoShowPass)
  {
    if ("						" == frm_Password)
    {
      Password = $("#RealPassword").val();
    }
    else
    {
      Password = frm_Password;
    }
  }
  else
  {
    Password = frm_Password;
  }
  var SHA256Password = sha256(Password + xmlObj);
  var LoginFormObj = new webSubmitForm();
  LoginFormObj.addParameter("Username", $("#Frm_Username").val());
  LoginFormObj.addParameter("Password", SHA256Password);
  LoginFormObj.addParameter("action", "login");
  LoginFormObj.addParameter("_sessionTOKEN", "820326693287295952528132");
  LoginFormObj.Form.submit();
  Password = undefined;
  SHA256Password = undefined;
}
```

👀 There it was.

The secret exposed itself:  

```javascript
var SHA256Password = sha256(Password + xmlObj);
```

---

### 🛜 My Immediate Thoughts

> *"Wait... so it's NOT just hashing the password..."*  
> *"It's combining the password with some other thing... this mysterious `xmlObj`!"*

**Password** ➕ **xmlObj** ➔ **SHA256** ➔ Final password sent.

*No colons, no spaces, no newlines - just directly glued together.*

---

### 🌙 But What the Hell is `xmlObj`?

The code showed:

```javascript
var xmlObj = $(xml).text();
```

Meaning:

- `xmlObj` came from somewhere *outside* - some dynamic input.
- Probably an AJAX server response - like a hidden login token sent quietly behind the scenes.

---

## Part 4 - Finding the True XML Token in the Network

At this point, I knew the password hashing wasn't magic.  
It was **password + xmlObj**, glued together, and then SHA-256 hashed.

But...  
where the hell was this **`xmlObj`** actually *coming from*?

Time to hack deeper. 🥷

---

### 🛜 My Plan: Catch the Token in Live Traffic

1. Opened **DevTools** → **Network tab**.
2. Reloaded the router login page.
3. Checked "**Preserve Log**" to make sure requests stayed visible after reload.
4. Set filter to **XHR** (only AJAX requests).

👉 My goal:  
Catch every little server whisper that the page pulls in after loading.

---

### 🍬 What I Found

I didn't find "loginToken" directly by sight...  
So I hit `Ctrl+F` inside the Network panel - **searched for**:

> **loginToken**

And then...  
✨ I found a request to:

```
/function_module/login_module/login_page/logintoken_lua.lua
```

---

### 🌸 The Response Revealed:

Opening its Response tab, I saw:

```xml
<ajax_response_xml_root>63794643</ajax_response_xml_root>
```

👀 There it was.  
**The real dynamic token.**

---

### 🛜 Final Understanding:

So the login process was:

1. Router sends a dynamic **small XML token** (`63794643`) secretly on page load.
2. Browser JavaScript (`g_loginToken(xml)`) catches it.
3. Password + XML token → concatenated without any separator.
4. Entire string SHA-256 hashed.
5. That hash is sent in the login `POST`.

---

### 🌙 I Rebuilt the Final Hash Manually:

- My password: `admin`
- The live token: `63794643`

Concatenated:
```
admin63794643
```
Hashed with SHA-256 ➔  
Result:

```
5fd0828219e305d95e21e4925c42faa32970e3702bcf4a23863133e5f7574b39
```

And when I compared it to the `POST` request password field?  
🎯 **It matched perfectly.**

---

## Part 5 - Final Checkpoints: Untangling Session Tokens and XML Token

At this point, I had fully cracked how the **password hash** was generated.

But...  
another tiny mess was sitting in front of me -  
the **_sessionTOKEN**.

---

### 🛜 What Confused Me

I noticed that:

- In the **page source**, there were hidden `<input>` fields like:

  ```html
  <input type="hidden" name="_sessionTOKEN" id="_sessionTOKEN" value="841955577041466843486997"/>
  <input type="hidden" name="_vueSessionTOKEN" id="_vueSessionTOKEN" value="841955577041466843486997"/>
  ```

- And in the **JavaScript**, there was also:

  ```javascript
  var g_sessionTmpToken = '841955577041466843486997';
  ```

These tokens looked similar but slightly suspicious.

---

### 🌸 Another Layer of Dynamic Tokens?

When I watched the **Network** tab carefully,  
I noticed:

- Every time I reloaded the page, the `_sessionTOKEN` value **changed**.  
- But sometimes, during login POST, the token inside the request body wasn't exactly matching the page's token when the page first loaded.

😵 Was there some AJAX refreshing the token *again* during login?

---

### 🛜 So What I Understood:

- The **xml token** I captured (`63794643`) is **only used for password hashing**.  
- The **_sessionTOKEN** is **used for server-side session validation** when submitting the login.

They are **different roles**:

| Token | Purpose |
|:---|:---|
| XML Token (from logintoken_lua.lua) | Used to build the hashed password |
| _sessionTOKEN (from hidden fields) | Sent separately for server validation |

---

### 🌙 What Really Happens During Login

1. Browser loads login page.
2. Receives **XML token** via AJAX (`logintoken_lua.lua`).
3. JavaScript concatenates `password + XML token`, hashes it via SHA-256.
4. Browser fills in:
   - `Username`
   - `Password` (hashed version)
   - `_sessionTOKEN` (latest one)
   - `action = login`
5. Sends a `POST` request.

If any part is wrong - wrong hash, wrong session token - router login **fails**.

✅  
If everything matches - login **success**.

---

### 🍬 My Final Confirmation

When I manually re-built:

```plaintext
"admin" + "63794643"
```

→ SHA256 → `5fd0828219e305d95e21e4925c42faa32970e3702bcf4a23863133e5f7574b39`

Then sent along with the correct `_sessionTOKEN` captured from hidden input field...

🎯 **Router accepted the login.**

I didn't even need the real login page anymore if I scripted it myself. 😈

---

## Part 6 - Summary: How to Fully Crack a Router's Login Process

If you ever find yourself facing a router login page using **HTTP** (not HTTPS),  
and notice that your password field in the `POST` request doesn't match your raw password,  
here's how **you** can fully reverse-engineer the process.

---

### 📋 Step 1 - Connect and Open DevTools

- Connect to the router (via WiFi or Ethernet).
- Open your browser and visit the router login page (e.g., `http://192.168.1.1`). 
- Open **Developer Tools** (`F12` or `Ctrl+Shift+I`).
- Switch to the **Network** tab.
- **Enable "Preserve Log"** to keep network traffic even after reload.

#### 🍬 Tiny Tips:

**Can't get in?** Type `ipconfig` in your Windows CMD (or `ifconfig` in Linux/Mac Terminal) - find your **Default Gateway** - that's your router's true IP address!

---

### 📋 Step 2 - Filter and Find Token Requests

- Set the Network filter to **"XHR"** (AJAX requests).
- Reload the page.
- Search (`Ctrl+F`) for keywords like **`token`**, **`loginToken`**, or similar.
- Look for any small request that loads a dynamic value from the server.

---

### 📋 Step 3 - Capture the Live XML Token

- Open the **Response** tab of suspicious requests.
- Find XML like:
  ```xml
  <ajax_response_xml_root>63794643</ajax_response_xml_root>
  ```
- **Note the token value** (`63794643` in this example).

---

### 📋 Step 4 - Check the JavaScript Source

- Switch to the **Sources** tab.
- Search (`Ctrl+Shift+F`) for keywords: **`password`**, **`hash`**, **`sha256`**, **`encrypt`**.
- Find the login-related function (often something like `g_loginToken(xml)`).
- Analyze how the password is processed.

Common pattern:

```javascript
var SHA256Password = sha256(Password + xmlObj);
```

Meaning:

- **Password** + **Token** (concatenated directly)
- Then **SHA-256 hashed**.

---

### 📋 Step 5 - Rebuild the Hashed Password

- Concatenate your password and the captured XML token **with no separator** (no spaces, no colons).
  ```
  admin63794643
  ```

- Hash the result using **SHA-256**.

Example output:

```
5fd0828219e305d95e21e4925c42faa32970e3702bcf4a23863133e5f7574b39
```

- Confirm that this hash matches the Password field you saw in the `POST` login request.

---

### 📋 Step 6 - Understand the _sessionTOKEN Field

- Besides the hashed password, your router likely requires a `_sessionTOKEN`.
- This is usually loaded as a hidden `<input>` field inside the login page.
- Make sure you send the correct, latest `_sessionTOKEN` alongside your username and hashed password.

---

### 📋 Step 7 - Craft Your Own Manual Login (Optional)

Now that you know the process,  
you can:

- Auto-fetch the token.
- Hash your password manually.
- Submit a crafted `POST` request without using the original web UI.

---

### 🌟 Final Thoughts

- Most consumer routers use simple HTTP login mechanisms with **token-based password hashing**.
- By understanding the JavaScript and the network traffic,  
  **you can fully reverse-engineer** the authentication logic.
- Be cautious:  
  hacking your own hardware is a beautiful skill -  
  but always respect the legal boundaries when targeting anything beyond your own devices.

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
