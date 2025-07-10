---
layout: post
title: "How ADHD Gamer Play PowerWash"
description: free up a finger by Using xdotool in Linux.
date: 2025-05-10 02:58:00 +0800
categories: [🤖 tech, 🐧 Linux]
tags: [🐧 Linux, 🎮 Steam, 🔫 PowerWash, 🦾 xdotool]
img_path: /assets/img/posts/
toc: true 
comments: true 
image: /assets/img/posts/power_wash.png
---

## 👾 use `xdotool` to simulate `TAB`

So looks like someone's got ADHD but have to play PowerWash. 🤷‍♀️

Let's make sure we can press `TAB` automatically every 8 seconds. Here's a simple way to do it with a script on Linux.

You can use `xdotool`, a tool that simulates keyboard input. Install it with:

```bash
sudo apt-get install xdotool
```

Once that's installed, create a script to press `TAB` every 8 seconds. 

1. Create a new script file:

```bash
nano press_tab.sh
```

2. Paste this into the file:

```bash
#!/bin/bash
while true
do
    xdotool key Tab
    sleep 8
done
```

3. Save and exit the editor (press `Ctrl+X`, then `Y`, then `Enter`).

4. Make the script executable:

```bash
chmod +x press_tab.sh
```

5. Run the script:

```bash
./press_tab.sh
```

Now, every 8 seconds, it will press `TAB` for you! 🟢 You can stop the script by pressing `Ctrl+C` when you're done.

---

## 👻 Use `nohup` to run it in the background

We can make this run without needing to keep the terminal open and make it less resource-hungry by running it as a background service.

To make it less intrusive and run in the background without occupying much of the terminal, we can use `nohup` (no hangup), which allows the script to run even after you close the terminal. Here's how you can set it up:

1. **Prepare the script**:

Make sure your script is ready (you should already have `press_tab.sh` from earlier). 

2. **Run the script in the background**:

Use `nohup` to run it in the background:

```bash
nohup ./press_tab.sh &
```

This will run the script in the background and keep it going even after you close the terminal. The `&` ensures that it runs in the background, and `nohup` makes sure it doesn't stop when the terminal closes.

## 🔪 Check and Stop the script

You can check if it's running by looking for the `press_tab.sh` process with:

```bash
ps aux | grep press_tab.sh
```

Stopping the script is easy! You have a couple of options:

### 🙅‍♀️ 1. **Using `ps` to find the process and kill it**:

If you ran the script using `nohup` or in the background, you can kill it by finding the process ID (PID) and using the `kill` command.

1. **Find the PID**:
   Open a terminal and run:

   ```bash
   ps aux | grep press_tab.sh
   ```

   You should see something like this:

   ```
   your_user_name   12345  0.0  0.1  123456  7890 ?  S    10:00   0:00 ./press_tab.sh
   ```

   The number in the second column (`12345` in this example) is the PID (Process ID).

2. **Kill the process**:
   To stop the script, run:

   ```bash
   kill 12345
   ```

   Replace `12345` with the actual PID number you found.

If you want to forcefully kill it (just in case), you can use:

```bash
kill -9 12345
```

### 👩‍⚖️ 2. **If it's running as a background job with `nohup`**:

If you're running the script using `nohup` and have it in the background, you can just use `jobs` to find the job ID and then kill it.

1. **List the jobs**:

```bash
jobs
```

You'll see something like:

```
[1]+  12345  Stopped                 ./press_tab.sh
```

2. **Kill the job**:

```bash
kill %1
```

Here `%1` refers to the job number (in this case, `1`). You can also use `kill %n`, where `n` is the job number.

Once the script is killed, it will stop pressing `TAB` for you. 🛑

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
