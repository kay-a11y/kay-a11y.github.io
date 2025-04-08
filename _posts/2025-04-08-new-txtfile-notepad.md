---
layout: post
title: "[Windows11] Add `New > text file ` to Context Menu"
date: 2025-04-08 17:39:00 +0800
categories: [🤖 tech, ⚙️ setup]
tags: [⚙️ setup, 🌌 environment, 🪟 Windows, 🟩 registry editor]
img_path: /assets/img/posts/ 
toc: true 
comments: true 
---

<iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/72riifJHqCjI47UJkUllUZ?utm_source=generator" width="100%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>

## 😤 The Nightmare Begins: No New > Text File Option

It all started when I 🔪 uninstalled Notepad and replaced it with Notepad++ 💪. Everything was fine until I realized I couldn't create a simple `.txt` file from the right-click context menu anymore. Like, **seriously**?! 😑

Turns out, the entire registry structure for `.txt` files got broken or messed up 💦. And of course, I just had to fix it. Because, why not? 😈

## 😮‍💨 Solution: Fixing The Mess

💡 Here's how to **bring back `New > Text File` to Windows 11 context menu**.

🔗 [**Reference**](https://www.tenforums.com/tutorials/24412-add-remove-default-new-context-menu-items-windows-10-a.html)

The "Restore_New_Text_Document_context_menu_item.reg" file worked fine for *Win11*, though the tutorial highlighted it was for Win10.

### 📖 Steps to Restore The Option

1. 💾 Download the `Restore_New_Text_Document_context_menu_item.reg` file from the link above.

2. 📂 Save it to your desktop.

3. 🖱️ Double-click on the downloaded `.reg` file to **merge it with your registry**.

4. When prompted, click/tap on Run, ✅ Yes (UAC), ✅ Yes, and ✅ OK to approve the merge.

5. Hit `Ctrl + Shift + Esc` > Restart `explorer.exe`

6. 🚮 You can now delete the downloaded `.reg` file if you like.

## 📋 Registry Code Backup

Here's the full `.reg` file I used. Just in case you need to do it manually (because who doesn't love getting their hands dirty in the registry? 😜):

```reg
Windows Registry Editor Version 5.00

; Created by: Shawn Brink
; Created on: September 28th 2015
; Updated on: August 28th 2019
; Tutorial: https://www.tenforums.com/tutorials/24412-add-remove-default-new-context-menu-items-windows-10-a.html


; Text Document
[-HKEY_CLASSES_ROOT\.txt\ShellNew]
[HKEY_CLASSES_ROOT\.txt\ShellNew]
"ItemName"=hex(2):40,00,25,00,53,00,79,00,73,00,74,00,65,00,6d,00,52,00,6f,00,\
  6f,00,74,00,25,00,5c,00,73,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,\
  00,6e,00,6f,00,74,00,65,00,70,00,61,00,64,00,2e,00,65,00,78,00,65,00,2c,00,\
  2d,00,34,00,37,00,30,00,00,00
"NullFile"=""


[-HKEY_CLASSES_ROOT\.txt]

[HKEY_CLASSES_ROOT\.txt]
@="txtfile"
"Content Type"="text/plain"
"PerceivedType"="text"

[HKEY_CLASSES_ROOT\.txt\PersistentHandler]
@="{5e941d80-bf96-11cd-b579-08002b30bfeb}"

[HKEY_CLASSES_ROOT\.txt\ShellNew]
"ItemName"=hex(2):40,00,25,00,53,00,79,00,73,00,74,00,65,00,6d,00,52,00,6f,00,\
  6f,00,74,00,25,00,5c,00,73,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,\
  00,6e,00,6f,00,74,00,65,00,70,00,61,00,64,00,2e,00,65,00,78,00,65,00,2c,00,\
  2d,00,34,00,37,00,30,00,00,00
"NullFile"=""

[-HKEY_CLASSES_ROOT\SystemFileAssociations\.txt]

[HKEY_CLASSES_ROOT\SystemFileAssociations\.txt]
"PerceivedType"="document"

[-HKEY_CLASSES_ROOT\txtfile]

[HKEY_CLASSES_ROOT\txtfile]
@="Text Document"
"EditFlags"=dword:00210000
"FriendlyTypeName"=hex(2):40,00,25,00,53,00,79,00,73,00,74,00,65,00,6d,00,52,\
  00,6f,00,6f,00,74,00,25,00,5c,00,73,00,79,00,73,00,74,00,65,00,6d,00,33,00,\
  32,00,5c,00,6e,00,6f,00,74,00,65,00,70,00,61,00,64,00,2e,00,65,00,78,00,65,\
  00,2c,00,2d,00,34,00,36,00,39,00,00,00

[HKEY_CLASSES_ROOT\txtfile\DefaultIcon]
@=hex(2):25,00,53,00,79,00,73,00,74,00,65,00,6d,00,52,00,6f,00,6f,00,74,00,25,\
  00,5c,00,73,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,00,69,00,6d,00,\
  61,00,67,00,65,00,72,00,65,00,73,00,2e,00,64,00,6c,00,6c,00,2c,00,2d,00,31,\
  00,30,00,32,00,00,00

[HKEY_CLASSES_ROOT\txtfile\shell\open\command]
@=hex(2):25,00,53,00,79,00,73,00,74,00,65,00,6d,00,52,00,6f,00,6f,00,74,00,25,\
  00,5c,00,73,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,00,4e,00,4f,00,\
  54,00,45,00,50,00,41,00,44,00,2e,00,45,00,58,00,45,00,20,00,25,00,31,00,00,\
  00

[HKEY_CLASSES_ROOT\txtfile\shell\print\command]
@=hex(2):25,00,53,00,79,00,73,00,74,00,65,00,6d,00,52,00,6f,00,6f,00,74,00,25,\
  00,5c,00,73,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,00,4e,00,4f,00,\
  54,00,45,00,50,00,41,00,44,00,2e,00,45,00,58,00,45,00,20,00,2f,00,70,00,20,\
  00,25,00,31,00,00,00

[HKEY_CLASSES_ROOT\txtfile\shell\printto\command]
@=hex(2):25,00,53,00,79,00,73,00,74,00,65,00,6d,00,52,00,6f,00,6f,00,74,00,25,\
  00,5c,00,73,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,00,6e,00,6f,00,\
  74,00,65,00,70,00,61,00,64,00,2e,00,65,00,78,00,65,00,20,00,2f,00,70,00,74,\
  00,20,00,22,00,25,00,31,00,22,00,20,00,22,00,25,00,32,00,22,00,20,00,22,00,\
  25,00,33,00,22,00,20,00,22,00,25,00,34,00,22,00,00,00

[-HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.txt]

[HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.txt\OpenWithList]

[HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.txt\OpenWithProgids]
"txtfile"=hex(0):

[HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.txt\UserChoice]
"Hash"="hyXk/CpboWw="
"ProgId"="txtfile"

[-HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\Roaming\OpenWith\FileExts\.txt]

[HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\Roaming\OpenWith\FileExts\.txt\UserChoice]
"Hash"="FvJcqeZpmOE="
"ProgId"="txtfile"
```

## 🎉 Conclusion

After merging the `.reg` file, my beautiful `New > Text File` option was finally back! 🙌

If you're stuck in the same nightmare, hope this helps you escape. 🔥