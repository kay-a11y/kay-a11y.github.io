---
layout: post
title: "[HexChat][Windows] A Feasible Way to Achieve a Full Dark Theme (Including GTK UI)" 
date: 2025-03-30 10:48:00 +0800 
categories: [🤖 tech, ⚙️ setup] 
tags: [🌴 environment, ⚙️ setup, 🗣️ IRC, 🪟 Windows, 🗣️ HexChat, 🌚 dark mode] 
toc: true 
comments: true 
---

<iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/4IT6vDuKprKl6jyVndlY8V?utm_source=generator" width="100%" height="152" frameBorder="0" allowfullscreen="" allow="clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>

<br>
<p> 
  <img 
    src="/assets/img/posts/hexchat-logo.svg"
    alt="hexchat logo"
    style="
      display: block;
      margin-left: auto;
      margin-right: auto;
      width: 100px;
      height: auto;
    "
  />
</p>

I'm quite particular about having a consistent dark mode everywhere. I found the official HexChat documentation's method for application-specific GTK themes (`https://hexchat.readthedocs.io/en/latest/appearance.html#gtk-theme`) didn't work for my setup (HexChat 2.16.2 on Windows 11). The solution was to set a **global** GTK theme instead. Here's how I got it working:

## 🎨 Step 1: Theming the Chat Content Area (colors.conf)

This step themes the main chat text, user list, input bar, etc., but **not** the window borders or menus.

1.  Download a `colors.conf` file from a theme repository like [Catppuccin for HexChat](https://github.com/catppuccin/hexchat). (Choose a theme you like!)
2.  Go to your HexChat configuration directory by typing `%APPDATA%\HexChat\` into the File Explorer address bar and pressing Enter.
3.  Replace the existing `colors.conf` file in this directory with the one you downloaded.

## 🛠️ Step 2: Theming the Application Window & Menus (Global GTK Theme)

This step themes the "outer shell" of HexChat – the window frame, menus, buttons, etc., making it truly dark.

### 🖌️ 2.1 Download a Compatible GTK2 Theme

*   You need a **GTK2** theme that uses the **Pixbuf engine**, as this is what HexChat for Windows typically supports well.
*   The classic [Vertex theme](https://github.com/horst3180/vertex-theme/releases) is known to work (I used `vertex-theme-20170128.zip` successfully). You can find the original download there, or look for more modern forks/alternatives if you prefer.

### 🔧 2.2 Install the Theme Globally for Your User

1.  **Unzip** the downloaded theme archive (e.g., `vertex-theme-20170128.zip`) to a temporary location.
2.  Navigate inside the unzipped folder, typically to a path like `vertex-theme\common\gtk-2.0-dark\` (the exact path might vary, look for the `gtk-2.0` directory corresponding to the dark variant).
3.  **Copy all the contents** from inside this `gtk-2.0-dark` directory.
4.  Go to your user profile directory by typing `%USERPROFILE%` into the File Explorer address bar and pressing Enter (this usually opens `C:\Users\<YOUR_USER_NAME>`).
5.  Inside your user profile directory, create a new folder named `.themes` (note the leading dot) if it doesn't already exist.
6.  Open the `.themes` folder and create a new folder inside it named `vertex` (or whatever your theme is called).
7.  **Paste** the theme contents you copied in step 3 into this new `vertex` folder. The final path to the theme's `gtkrc` file should look something like `C:\Users\<YOUR_USER_NAME>\.themes\vertex\gtk-2.0\gtkrc`.

### ✒️ 2.3 Configure GTK to Use the Theme

1.  Go back to your main user profile directory (`%USERPROFILE%`).
2.  Create a new file named `.gtkrc-2.0` (note the leading dot and no file extension) if it doesn't already exist. **Make sure it doesn't end with `.txt`!** (In Notepad, save as "All Files" and type the name in quotes: `".gtkrc-2.0"`).
3.  Open this `.gtkrc-2.0` file with a plain text editor (like Notepad, Notepad++, VS Code, etc.).
4.  Add the following single line:
    ```
    gtk-theme-name = "vertex"
    ```
    (Replace `"vertex"` if you used a different theme folder name in step 2.6).
5.  Save the `.gtkrc-2.0` file.
6.  **Restart HexChat.** It should now launch with the full dark theme applied to both content and window elements!

## ❗ Important Considerations

*   **Affects All GTK2 Apps:** This global method will change the appearance of **all** GTK2 applications running under your user account on Windows (though there might not be many common GTK2 apps these days besides HexChat).
*   **Why Global Works:** This is the standard way GTK2 applications look for theme configurations on Unix-like systems, and it seems HexChat for Windows recognizes this global setting more reliably than the application-specific one placed in its installation directory.

Hope this helps others achieve that perfect dark mode HexChat setup! Have fun!

## 😾 ROAST - stop cursor from blinking PLS

HexChat has a super annoying bug.

The blinking cursor really distracts me (maybe that just proves I kind of have ADHD, lol). 

The issue is, you cannot apply both the theme and cursor settings in .gtkrc-2.0 simultaneously, even though you'd expect something like this to work (which seems totally legit):

```
gtk-theme-name = "vertex"
gtk-cursor-blink = false
gtk-cursor-blink-time = 0

```

You can either get your eye-friendly dark mode using just this:

```
gtk-theme-name = "vertex"
```

OR just stop the blinking with this:

```
gtk-cursor-blink = false
gtk-cursor-blink-time = 0

```

This 'either/or' limitation was a deal-breaker for me. I need my dark theme and no damn blinking cursor! That's why I switched and am now configuring AdiIRC.

