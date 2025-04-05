---
layout: post
title: "[Okular][Windows] Permanent dark mode and Language Switch"
date: 2025-04-05 08:52:00 +0800
categories: [🤖 tech, ⚙️ setup]
tags: [👶🏻 beginner, ⚙️ setup, 🌌 environment, 🌚 dark mode, 📖 pdf reader, 📚 Okular, 🖥️ bash]
img_path: /assets/img/posts/ 
toc: true 
comments: true 
---

<iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/76NX6bt1KCSg0yULpmlkyJ?utm_source=generator" width="100%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>

## 🕶️ Dark Mode

### 🐈‍⬛ Switch to 🌚 Dark Mode RIGHT NOW!

Basically, we switch 🌚 dark mode like this:

⚙️ `Settings` > ⚙️ `Configure Okular` > ✅ tick `change colors` > 🌙 `color mode: change dark & light colors` > 🎨 choose 2 colors (🌈 Colors in the gif are:   #cec5b4 & #2d2d2d)

![switch_dark.gif](/assets/img/posts/switch_dark.gif)

### 🦉 Permanent Dark Mode

But the thing is, this setting is only temporary. If you exit the app and restart it, it'll back to ☀️ light mode again!

So here's how to make 🌚 dark mode permanent.

* 🔔 **Important:** I downloaded **Okular** from the Microsoft Store and moved it to my 💾 `E:/`.

1. 📁 Check this path `C:\Users\YourUsername\AppData\Local\okular\okularpartrc
`
    * If there's a file named `okularpartrc`, ✅ great. If not, just create it here. (No `.txt` extension, just plain `okularpartrc`)

2. ✍️ **Edit or Create the `okularpartrc` file:**

    ```ini
    [Rendering]
    ChangeColors=true
    ColorMode=2
    DarkColor=#2D2D2D  ; 🌚 Your preferred dark background color
    LightColor=#CEC5B4  ; 🌙 Your preferred text color
    ```

    💾 **Save the file** and restart **Okular**.🔄 If it doesn't work, you may need to **reset the two colors manually again** and click 💾 `File > Save` just in case.

    Restart again. This time should be dark perfectly.

## 🔤 Change Language (e.g. `en_US`)

### 🛠️ Why Changing Language via UI Doesn't Work

There is a language change issue on Windows:

In `⚙️ Setting > Configure Language - Okular`, I can't change the main language to English. When I try to change and click save, it said will change when reboot. But when I restart, it still keeps the old language setting, no matter how many times I try to change it.

So I guess here's the thing, once you choose one language in the process of installation, **Okular** from the Microsoft Store seems to lock that setting. 🔒.

### 🛠️ 2 Ways to Change Language

#### 🖥️ **Forcing Language Change Through Environment Variables**

1. **Press `Win + R`**, type `sysdm.cpl`, and hit **Enter**. This opens the 🖥️ **System Properties** window.

2. Click on the ⚙️ **Advanced** tab, then click on 🔍 **Environment Variables**.

3. In the 🖥️ **System variables** section, click on ➕ **New...**.

4. 📝 **Create a New Variable:**
   - **Variable name:** `LANG`
   - **Variable value:** `en_US`

5. Click ✅ **OK** to save, then ✅ **OK** again to close all windows.

6. 🔄 **Restart your computer** (I managed to do it just by restarting Okular.).

7. **Open Okular** and see if it starts in **English**.

#### 💻 **Batch File Method (💪 Full Control Over Okular Launch)**

Instead of trying to pass the environment variable through a shortcut directly, we'll make a **batch file** to **launch Okular** in English mode.

1. 📑 **Open Notepad or Notepad++** (or whatever text editor you love)

2. **Paste the following code:**

    ```batch
    @echo off
    set LANG=en_US
    start "" "path\to\your\okular.exe" # your path to okular.exe
    ```

    - **Explanation:**
    - `@echo off`: Hides the command prompt details.
    - `set LANG=en_US`: Sets the language to English **temporarily**.
    - `start`: Launches Okular in a **new process** with the `LANG` environment variable applied.

3. 💾 **Save the file as a `.bat` file:**  
    - **File Name:** `LaunchOkularEnglish.bat`  
    - **Location:** Save it wherever you want, but maybe keep it in the same folder as your Okular executable.

4. 📂 **Double-click** the `LaunchOkularEnglish.bat` file.  

5. **Okular should now launch in English**, regardless of your system's language settings.


##### 🌟 **Making It Fancy**

If you want, you can **create a shortcut for this batch file** and place it on your desktop or taskbar for easy access.

**Why This Works:**  
The batch file **temporarily sets the `LANG` environment variable** just for launching Okular. No impact on your other apps, no messing with global variables. Just pure, focused execution.