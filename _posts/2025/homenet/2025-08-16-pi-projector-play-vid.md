---
layout: post
title: "Play Cam video on Pi via projector"
description: 
date: 2025-08-16 01:19:00 +0000
categories: [🤖 tech, 🐧 Linux]
tags: [🐧 Linux, 🔀 LAN, 🍓 Raspberry Pi]
img_path: /assets/img/posts/
math: true
toc: true 
comments: true 
image: 
---

## Goal

Set up a 2×2 video wall using the Raspberry Pi, connected to a projector via HDMI, and then project it onto the wall. The easiest path on a Pi is to let **FFmpeg** tile the 4 RTSP feeds, and pipe to **mpv**.

## Test on PC first

### Use mpv (ffmpeg → mpv)

**Quick test**:

```bash
mpv --no-audio --profile=low-latency \
    --demuxer-lavf-o=rtsp_transport=tcp \
    rtsp://admin:PASS@Cam_IP:554/Streaming/Channels/102
```

**2×2 video wall**:

```bash
ffmpeg -fflags nobuffer -flags low_delay -rtsp_transport tcp \
 -i "rtsp://admin:PASS@CAM1_IP/Streaming/Channels/102" \
 -i "rtsp://admin:PASS@CAM2_IP/Streaming/Channels/102" \
 -i "rtsp://admin:PASS@CAM3_IP/Streaming/Channels/102" \
 -i "rtsp://admin:PASS@CAM4_IP/Streaming/Channels/102" \
 -filter_complex "[0:v]scale=960:540[v0];[1:v]scale=960:540[v1];[2:v]scale=960:540[v2];[3:v]scale=960:540[v3];\
 [v0][v1][v2][v3]xstack=inputs=4:layout=0_0|960_0|0_540|960_540,format=yuv420p" \
 -f matroska - | mpv --no-audio --fullscreen -
```

### Performance Setting

* Pi 4/5 is fine with 4× 720p H.264; 4× H.265 may choke. If needed, drop each `scale` to `640:360`.
* Prefer **wired** Ethernet; Wi-Fi + 4 RTSP feeds can stutter.

### Shell Function

```bash
nvrsub() {
  ffmpeg -hide_banner -loglevel warning -rtsp_transport tcp \
    -thread_queue_size 512 -i "rtsp://admin:PASS@CAM1_IP:554/Streaming/Channels/102" \
    -thread_queue_size 512 -i "rtsp://admin:PASS@CAM2_IP:554/Streaming/Channels/102" \
    -thread_queue_size 512 -i "rtsp://admin:PASS@CAM3_IP:554/Streaming/Channels/102" \
    -thread_queue_size 512 -i "rtsp://admin:PASS@CAM4_IP:554/Streaming/Channels/102" \
    -filter_complex "[0:v]scale=640:360[v0];[1:v]scale=640:360[v1];[2:v]scale=640:360[v2];[3:v]scale=640:360[v3];\
[v0][v1][v2][v3]xstack=inputs=4:layout=0_0|640_0|0_360|640_360,format=yuv420p" \
    -f matroska - | mpv --no-audio --fullscreen --profile=low-latency --hwdec=auto-safe -
}
```

Save and reload: `source ~/.zshrc`.

Use:

```bash
nvrsub
```

## Hands on Raspberry Pi

### Install on Raspberry Pi OS (Bookworm/Bullseye)

```bash
sudo apt update
sudo apt install ffmpeg mpv
```

Extra niceties:

```bash
sudo apt install libavcodec-extra      # widest codec set
sudo apt install fonts-dejavu-core     # if you'll use drawtext labels
```

### Test GUI on Pi by SSH

**SSH X11 forwarding**: run the app on the Pi, window pops up on PC.

#### Fast path (works on Linux PC)

**SSH to Pi:**

```bash
sudo apt update
sudo apt install xauth x11-apps   # xeyes/xclock for testing
# (optional) sudo apt install mesa-utils  # glxinfo/glxgears tests
```

Make sure SSH allows X11:

```bash
sudo nano /etc/ssh/sshd_config
```

Set (or add):

```bash
X11Forwarding yes
X11UseLocalhost yes
```

Then:

```bash
sudo systemctl restart ssh
```

**Open another session, from PC → Pi:**

```bash
ssh -X -C pi@<pi-ip>    # try -Y if some apps complain
echo $DISPLAY           # should print something like: localhost:10.0
xeyes &                 # or xclock/xcalc - a window should appear on your PC
```

Now launch any Pi GUI app the same way (from that SSH session).

Tip: X11 forwarding is okay for **light** GUIs. Don't try video playback this way; use ffmpeg/mpv pipeline instead.

That little eyeball is **xeyes**. It's a tiny X11 demo that **follows your mouse**, and it proves X11 forwarding is working: the app is **running on the Pi**, but the window is **drawn on your PC**.

* The `&` just runs it **in the background** so your shell prompt comes back.

  * See it with `jobs`
  * Bring it back: `fg %1`
  * Kill it: `kill %1` (or just close the window)

#### Use X11 forwarding for light GUIs

From your existing `ssh -X` or `-Y` session, try:

```bash
xclock &
xcalc &
nm-connection-editor &   # edit Wi-Fi/Ethernet (Pi OS Bookworm uses NM)
```

Good for config tools and simple apps. But **don't** stream video this way.

After this test, just exit the `ssh -X -C` session.

#### Launch GUI on the Pi's projector from your PC

That will pop the 2×2 mosaic **on the projector** (not PC).

1. Interactive (ssh first → then run GUI there)

    **From PC:**

    ```bash
    ssh pi@PI_IP
    ```

    **Now you're inside the Pi's shell (prompt shows something like pi@raspberry)**. Re-do this step first -> [shell function](#shell-function). Then:

    ```bash
    # 1 Grant once per boot - this edits the Pi's display permissions
    DISPLAY=:0 XAUTHORITY=/home/pi/.Xauthority xhost +SI:localuser:pi
    # 2 Launch a test GUI ON THE PI'S SCREEN
    DISPLAY=:0 XAUTHORITY=/home/pi/.Xauthority xeyes &
    # 3 Launch your camera wall ON THE PI'S SCREEN
    DISPLAY=:0 XAUTHORITY=/home/pi/.Xauthority nvrsub
    ```

2. Make the grant automatic (systemd user unit)

    **From your PC (one shot to install on the Pi):**

    ```bash
    ssh pi@PI_IP 'mkdir -p ~/.config/systemd/user && cat > ~/.config/systemd/user/xhost-localuser.service <<EOF
    [Unit]
    Description=Allow local SSH user to access X/XWayland
    After=graphical-session.target
    PartOf=graphical-session.target

    [Service]
    Type=oneshot
    Environment=DISPLAY=:0
    Environment=XAUTHORITY=/home/pi/.Xauthority
    ExecStart=/usr/bin/xhost +SI:localuser:pi

    [Install]
    WantedBy=graphical-session.target
    EOF
    systemctl --user daemon-reload
    systemctl --user enable --now xhost-localuser.service'
    ```

    Now each time the Pi's desktop starts, the permission is granted.

    **Then, whenever you want to launch stuff (interactive):**

    ```bash
    ssh pi@PI_IP
    DISPLAY=:0 XAUTHORITY=/home/pi/.Xauthority xeyes &
    DISPLAY=:0 XAUTHORITY=/home/pi/.Xauthority nvrsub
    ```

    Or as a one-liner from another device (quotes keep it on the remote side):

    ```bash
    ssh pi@PI_IP \
    'DISPLAY=:0 XAUTHORITY=/home/pi/.Xauthority nvrsub'
    ```

##### Alias (optional)

Add this alias to **Pi's** `~/.bashrc`:

```bash
# draw GUI on the Pi's local screen from SSH
alias onpi='DISPLAY=:0 XAUTHORITY=/home/pi/.Xauthority '
```

Reload:

```bash
source ~/.bashrc
type onpi
```

Use:

```bash
onpi xeyes 
onpi nvrsub
```

> This works because the alias expands to:
> `DISPLAY=:0 XAUTHORITY=/home/pi/.Xauthority <your command here>`

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
