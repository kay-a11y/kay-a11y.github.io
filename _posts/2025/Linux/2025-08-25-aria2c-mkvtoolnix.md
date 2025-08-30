---
layout: post
title: "Continue disrupted downloads & strip audio tracks (aria2c + mkvtoolnix)"
description: 
date: 2025-08-25 09:02:28 +0000
categories: [🤖 tech, 🐧 Linux]
tags: [🐧 Linux, 🖥️ CLI, 📽️ Media, 🏴‍☠️ Pirate]
img_path: /assets/img/posts/
math: true
toc: true 
comments: true 
image: 
---

## Continue disrupted downloads with `aria2c`

A **browser download** doesn't save a proper "partial file" + "resume metadata" in the way `wget` or `curl` do. Firefox usually writes to something like `file.part`, but that piece alone doesn't carry the HTTP headers needed for resuming unless the server supports range requests *and* you know the exact same URL.

So while failed pirate a huge 64G total video on the browser, like with already downloaded 17G, ideally, we can continue that disrupted downloads with `aria2c`.

---

1. Grab the exact URL

    ```bash
    URL='https://your_download_link.mkv'
    # Test if ranged requests work (good = HTTP/1.1 206 + Content-Range)
    curl -L -s -D - -o /dev/null -r 0-0 "$URL" | grep -Ei 'HTTP/|Accept-Ranges|Content-Range'
    ```

    You can resume if you see **`206 Partial Content`** and **`Content-Range: bytes 0-0/…`**; If you only get `200 OK` with no `Accept-Ranges`, the server won't resume and any CLI will restart from 0.

2. Point the downloader at your partial file

    Find and rename Firefox's partial (usually ends with `.part`) to the *final* name you'll continue into:

    ```bash
    # Locate and inspect
    ls -lh
    # Suppose it's:
    mv "my_fav_movie_2160p_REMUX.part" "my_fav_movie_2160p_REMUX.mkv"
    ```

    Keep quotes if your filename has spaces/parentheses. Renaming is optional to make the continue flags nicer.

3. `aria2c` (fast and parallel for giant files)

    ```bash
    aria2c -c -x16 -s16 -k1M -o "my_fav_movie_2160p_REMUX.mkv" "$URL"
    ```

    - `-c` continue; `-x16 -s16` parallel connections; `-k1M` piece size hint.
    - If you already have the file with that exact name, it will pick up from ~17 GB.

## Strip Audio Tracks With `mkvtoolnix`

Some rips almost always include **multiple audio tracks**, and they love making the language I don't understand at all the default. We can strip audio tracks with `mkvtoolnix`.

1. **Check tracks with ffprobe / mediainfo**

   ```bash
   ffprobe -hide_banner "my_fav_movie_2160p_REMUX.mkv" | grep -i "Stream #.*Audio"
   ```

   You'll see something like:

    ```txt
    Stream #0:0: Video: hevc (Main 10), yuv420p10le(tv, bt2020nc/bt2020/smpte2084), 3840x2160 [SAR 1:1 DAR 16:9], 23.98 fps, 23.98 tbr, 1k tbn, 23.98 tbc (default)
    Stream #0:1(hin): Audio: ac3, 48000 Hz, stereo, fltp, 224 kb/s (default)
    Stream #0:2(eng): Audio: truehd, 48000 Hz, 7.1, s32 (24 bit)
    Stream #0:3(eng): Audio: ac3, 48000 Hz, 5.1(side), fltp, 448 kb/s
    ```

   (Order varies, but one will say `eng`, one `hin`.)

2. **Force English in VLC / MPV / etc.**
   - **VLC**:\
     `Audio → Audio Track → English`\
     Or set default: `Tools → Preferences → Audio → Preferred Audio Language → eng`.
   - **MPV**:\
     Add this line to your `~/.config/mpv/mpv.conf`:

     ```bash
     alang=eng
     ```

   - **Kodi**:\
     Settings → Player → Language → Preferred audio language = English.

## Strip Certain track (Optional)

1. Install the tools (MKVToolNix)

    - **Ubuntu**

    ```bash
    sudo apt update && sudo apt install mkvtoolnix mkvtoolnix-gui
    ```

    - **Arch**

    ```bash
    sudo pacman -S mkvtoolnix-cli
    ```

2. See exact MKV track IDs (MKVToolNix numbering)

    ```bash
    mkvmerge -i "my_fav_movie_2160p_REMUX.mkv"
    ```

    You'll get something like:

    ```txt
    Track ID 0: video (HEVC)
    Track ID 1: audio (AC-3) [hin]
    Track ID 2: audio (TrueHD/Atmos) [eng]
    Track ID 3: audio (AC-3 5.1) [eng]
    ```

    (Confirm IDs on your file, **these IDs are what mkvpropedit/mkvmerge use**, not ffprobe's `#0:1` style. If no **langs** had been tagged, just refer the output of `ffprobe`'s before.)

3. Keep both English tracks + make Atmos the default (no re-encode, instant edit)

    ```bash
    # Turn OFF Hindi as default
    mkvpropedit "my_fav_movie_2160p_REMUX.mkv" \
    --edit track:1 --set flag-default=0

    # Make English TrueHD/Atmos the default
    mkvpropedit "my_fav_movie_2160p_REMUX.mkv" \
    --edit track:2 --set flag-default=1
    --edit track:3 --set flag-default=0
    ```

    (If your IDs differ, adjust the numbers.)

    > To *remove* the Hindi track entirely but **keep both English**:
    >
    > ```bash
    > mkvmerge -o "my_fav_movie_2160p_EN.mkv" \
    >   -a 2,3 \
    >   "my_fav_movie_2160p_REMUX.mkv"
    > ```

4. Check afterwards

    ```bash
    mkvmerge -i "my_fav_movie_2160p_EN.mkv"
    ```

    Then:

    ```bash
    mkvinfo "my_fav_movie_2160p_EN.mkv" | grep -A5 "Track type: audio"
    ```

5. Check Which audio is on right now

    **mpv, live OSD:**

    Start mpv with a status line:

    ```bash
    mpv --term-status-msg='aid=${aid}  alang=${alang}  aname=${audio/codec-name}' \
        "my_fav_movie_2160p_REMUX.mkv"
    ```

    **mpv, log file way (very clear):**

    ```bash
    mpv --log-file=mpv.log "my_fav_movie_2160p_REMUX.mkv"
    grep -i 'aid=' mpv.log | tail -n 5
    ```

    You'll see lines like `aid=2 alang=eng` when it switches/loads.

    **ffprobe listing (static):**

    ```bash
    ffprobe -v error \
    -select_streams a \
    -show_entries stream=index,codec_name,channels:stream_tags=language,title \
    -of default=nw=1 "my_fav_movie_2160p_REMUX.mkv"
    ```

### Bonus: Rename the audio tracks (optional)

Rename the audio tracks with custom names:

```bash
mkvpropedit "my_fav_movie_2160p_EN.mkv" \
  --edit track:1 --set name="English TrueHD Atmos 7.1" \
  --edit track:2 --set name="English AC3 5.1 Fallback"
```

Run mpv with verbose status:

```bash
mpv --osd-level=3 --term-status-msg='aid=${aid} alang=${alang} aname=${audio}' "my_fav_movie_2160p_EN.mkv"
```

Now when you press `#` (multiple times), mpv will show something like:

```bash
aid=1 alang=eng aname=English TrueHD Atmos 7.1
aid=2 alang=eng aname=English AC3 5.1 Fallback
```

## Check if it has built-in subtitles

If `ffprobe`/`mkvmerge -i` output only showed **video + audio + chapters**, no subtitle streams were listed, meaning this particular REMUX **did not include subtitle tracks** (English subs, SDH, commentary, nothing). Release groups sometimes skip them to shave size, especially on these multi-lang rips.

We want to make sure 100% it doesn't have subtitles:

```bash
ffprobe -v error \
  -select_streams s \
  -show_entries stream=index:stream_tags=language,title \
  -of default=nw=1 "my_fav_movie_2160p_EN.mkv"
```

Or with mkvtoolnix:

```bash
mkvmerge -i "my_fav_movie_2160p_EN.mkv" | grep -i subtitle
```

If it prints **nothing**, then no subtitles are baked in.

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
