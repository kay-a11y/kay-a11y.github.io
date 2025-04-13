---
layout: post
title: "Converting PDFs to Markdown with Marker & Pandoc"
description: Set up with Marker and Pandoc
date: 2025-04-13 16:39:00 +0800
categories: [🤖 tech, ⚙️ setup]
tags: [⚙️ setup, 🐍 Python, 🪟 Windows, 📃 PDF Conversion, 🌐 Pandoc, 📝 Marker, 📚 Markdown, 💻 automation]
img_path: /assets/img/posts/
toc: true 
comments: true 
# image: /assets/img/posts/cypherpunk.png
---

<iframe style="border-radius:12px" src="https://open.spotify.com/embed/track/7dweUnOkEBtbYmGVEOiNLY?utm_source=generator" width="100%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>

## Marker → [GitHub](https://github.com/VikParuchuri/marker)

Let's get you set up with **Marker**, a powerful open-source tool for converting PDFs to Markdown. Here's a step-by-step guide to help you install and use it:

### 🛠️ Step 1: Install Dependencies

#### 1. **Install Python (Version 3.10 or er)**

- **Windows/macOS**: Download and install from the [official Python website](https://www.python.org/downloads/).
- **Linux**: Use your package manager. For example, on Debian/Ubuntu:

  ```bash
  sudo apt update
  sudo apt install python3 python3-pip
  ```

#### 2. **Install PyTorch**

- Visit the [PyTorch installation page](https://pytorch.org/get-started/locally/) and follow the instructions tailored to your system configuration.
- For CPU-only installation:

  ```bash
  pip install torch torchvision torchaudio
  ```
  
### 📦 Step 2: Install Marker

Once Python and PyTorch are set up, install Marker using pip:

```bash
pip install marker-pdf
```

### ✅ Step 3: Use `marker_single`

```bash
marker_single "E:/mydocs/my.pdf"
```

- This command will:
  - Process the full PDF (no page limit)
  - Output the `.md`, `.json`, and images in the virtual environment where you at now, like `E:/marker-env/.venv/Lib/site-packages/conversion_results/my`.

#### 🛠️ Optional flags you *can* use

```bash
--languages en
--max_retries 3
--max_table_rows 30
```

(You can check full list with `marker_single --help`)

### ✅ My Example: `CPU-only installation` in `virtual environment` on `E:/`

#### Two Qs to Clarify Before Installation

##### 🔍 What does "CPU-only" mean?

Most machine learning libraries like **PyTorch** come in two flavors:

- **CUDA version** (uses GPU for acceleration - needs NVIDIA GPU + drivers)
- **CPU-only version** (runs on your processor, slower but works everywhere)

If you are not using GPU like CUDA-enabled NVIDIA, the **CPU-only** install is the safest and most compatible option. 💻✅

##### 💾 Can you install it to `E:`?

Python and `pip` don't install packages *to a specific drive* like `E:\` by default - they install to the **Python environment** you're using. But yes, **you can control the installation location** with a **virtual environment** stored anywhere you like (like on E:).

#### 🔧 How to install Marker on `E:\` safely (step-by-step)

##### 1. Open CMD or PowerShell

```bash
E:
mkdir marker-env
cd marker-env
python -m venv env
.\env\Scripts\activate
```

💡 You're now inside a virtual environment *on E:*. Or you can just activate the virtual environment of your existing repo, similar to above.

##### 2. Install Marker (CPU-only style)

Now just run:

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
pip install marker-pdf
```

This installs **PyTorch (CPU-only)** and **Marker** right into your `E:\marker-env`.

##### 3. Test it

Try running for **a small pdf file**(like 5 pages):

```bash
marker_single "E:/mydocs/my.pdf"
```

That should spit out clean Markdown + any extracted images and metadata, all saved to your `E:/marker-env/.venv/Lib/site-packages/conversion_results/my` folder.

---

### Process PDF Manually Before Run `Marker`

Okay, so now we know how to use Marker to **process a small file**. But Once we want to process **an entire PDF book**, things can get *tricky*.

First thing first, `MemoryError`! Generally, if we let Marker the whole PDF once, there will be too much memory usage for system, and finally we'll get a **fail**.

So here's the *plan*, **based on the file input & output path**:

We’ll write a `marker_batch_run.bat` file that automates the following steps:

  1. Automatically slice PDF into 10-page chunks and save each one to a folder locally
  2. Loop through and run `marker_single` on each chunk(pdf)
  3. Move all generated folders to output path

Here's the script:

```bat
@echo off
setlocal enabledelayedexpansion

REM 🔧 Customize these paths
set "pdf_dir=E:\mydocs\my\sliced"
set "out_dir=E:\mydocs\my\markdown_output"
set "marker_out_dir=E:\marker-env\.venv\Lib\site-packages\conversion_results"

REM 🧙‍♂️ Activate virtual environment
call E:\marker-env\.venv\Scripts\activate.bat

REM 📁 Make output folder if missing
if not exist "!out_dir!" mkdir "!out_dir!"

echo 🔁 Starting batch conversion...

for %%F in ("%pdf_dir%\*.pdf") do (
    echo 🧾 Converting: %%~nxF
    marker_single "%%F"

    set "filename=%%~nF"
    set "source_folder=!marker_out_dir!\!filename!"
    set "target_folder=!out_dir!\!filename!"

    if exist "!source_folder!" (
        echo 🚚 Moving output folder: !source_folder! → !target_folder!
        move /Y "!source_folder!" "!target_folder!" >nul
    ) else (
        echo ⚠️  Output not found for %%~nxF — check Marker logs.
    )
)

echo ✅ All conversions complete!
pause
```

### Merge PDFs

#### Now we should've done in converting all `pdf chunks` to `markdown`.

It's time to *merge* them up, and *extract* all the images for a better embed with the complete `.md` file. Write a Python Script like below, which is gonna:

1. Scan your 1markdown_output/1 folder

2. For each folder (e.g., my_001-010), finds the .md file

3. Append each file's content to one big merged file (e.g., `my_001-471.md`)

4. Sort the folders in correct numeric order; Adds a nice header before each chapter block

5. Copy & rename *.jpeg files like `my_001.jpeg`, `my_002.jpeg`...

🧐 **Attention**: The image stuff may spend an extra time to check manually. It would be a boring job if you gotta a ton of them to process. 💦

#### Script For Merge & Extract

```py
import os
import glob
import shutil
from pprint import pprint

# 👇 Change this to your actual path
base_dir = r"E:\mydocs\my\markdown_output"
output_md_path = os.path.join(base_dir, "my_001-471.md")
image_output_dir = base_dir  # Output all .jpeg files here

# Get folders like my_001-010
folders = [f for f in os.listdir(base_dir) if os.path.isdir(os.path.join(base_dir, f)) and f.startswith("my_")]

# Sort them by the first number (e.g., 001, 011...)
def folder_sort_key(name):
    parts = name.replace("my_", "").split("-")
    return int(parts[0])

folders.sort(key=folder_sort_key)

# Merge markdowns and extract images
image_counter = 1
with open(output_md_path, "w", encoding="utf-8") as outfile:
    for folder in folders:
        folder_path = os.path.join(base_dir, folder)

        # 🔗 Merge .md file
        md_files = glob.glob(os.path.join(folder_path, "*.md"))
        if md_files:
            with open(md_files[0], "r", encoding="utf-8") as infile:
                outfile.write("\n\n---\n\n")
                outfile.write(infile.read())

        # 🖼 Copy & rename images
        image_files = sorted(glob.glob(os.path.join(folder_path, "*.jpeg")))
        pprint(image_files)
        for img in image_files:
            new_name = f"my_{image_counter:03d}.jpeg"
            new_path = os.path.join(image_output_dir, new_name)
            shutil.copy(img, new_path)
            image_counter += 1

print(f"✅ Merge complete: {output_md_path}")
print(f"✅ Total images extracted: {image_counter - 1}")
```

It's ALL DONE NOW! 🎊

## Pandoc - Converting documents between formats like PDF, EPUB, Markdown, and more

Let's get started with **Pandoc**, a tool for converting documents between formats like PDF, EPUB, Markdown, and more. Here's a step-by-step guide to help you master it:

---

### 💻 Step 1: Install Pandoc

#### 🪟 Windows

1. **Download Installer** Visit the [Pandoc Downloads page](https://pandoc.org/installing.html) and download the Windows installe.
2. **Run Installer** Double-click the downloaded `.msi` file and follow the installation prompt.

#### 🍎 macOS

1. **Download Package*: Go to the [Pandoc Downloads page](https://pandoc.org/installing.html) and download the macOS packae.
2. **Install*: Open the downloaded `.pkg` file and follow the installation instructios.

#### 🐧 Linux (Debian/Ubunu)

Open your terminal and un:
```bsh
sudo apt update
sudo apt install pado
```

---

### 🧪 Step 2: Verify Installation

After installation, confirm that Pandoc is installed correctly:

```ash
pandoc --version
```

You should see the installed version of Pandoc displayed.

---

### 🔄 Step 3: Basic Conversions

Pandoc can convert files between various formats. Here are some common examples:

#### 📄 Convert Markdown to HTML

```bash
pandoc input.md -o output.html
```

#### 📚 Convert Markdown to PDF

Note: Converting to PDF requires a LaTeX engine (e.g., TeX Live, iKTeX).

```bash
pandoc input.md -o output.pdf
```

#### 📘 Convert Markdown to EPUB

```bash
pandoc input.md -o output.epub
```

#### 📝 Convert Word Document to MarkdowN

```bash
pandoc input.docx -o output.md
```

#### 📄 Convert PDF t Markdown

Pandoc doesn't support PDF as an input format directly. For converting PDFs to Markdown, consider using tools like [Marker](https://github.com/VikParuchuri/marker) or other PDF to Markdown onverters.

---

### ⚙️ Step 4: Useful Options

- `-s` or `--standalone`: Produce a standalone document (with header ad footer).
- `-f` or`--from`: Specify input format (e.g., `markdown, `html`).
- `-t` r `--to`: Specify output format (e.g., `pdf, `epub`).
- `-o`: Specify output file name.

**Example*:
```bash
pandoc -s -f markdown -t html -o output.htl input.md
```

---

## 📚 Additional Resources

- **Official Documntation**: For more detailed information, visit the [Pandoc User's Guide](https://pandoc.org/MNUAL.html).
- **Tutorials**:
  - [Getting Started with Pandoc](https://pandoc.org/getting-started.html)
  - [How to Use Pandoc - An Open Source Tool for Technical Writers](https://www.freecodecamp.org/news/how-to-use-pandoc/)


<div class="donation-box" style="position: relative;">
  <p class="donation-text">💖 Support me with crypto or PayPal! 💘</p>
  <p><strong>Bitcoin (BTC):</strong><br>bc1qtzjwfyfpleyzmpqu97sdatqes98ms3zxc7u790</p>
  <p><strong>Ethereum (ETH) & USDT (ERC-20):</strong><br>0xFE05f74DeF594f8F904D915cB93361C99cB36500</p>
  <p>Or support me on Ko-fi:</p>
  
  <div class="img-container" style="position: relative; display: inline-block;">
    <!-- 图片 -->
    <img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png"
         alt="Support me on Ko-fi"
         width="150"
         loading="lazy">    
    <!-- 遮罩层按钮 -->
    <div onclick="window.open('https://ko-fi.com/kikisec', '_blank')" 
         style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: transparent; cursor: pointer;">
    </div>
  </div>

  <p class="donation-note">Any amount helps me continue creating content 💬💻</p>
</div>
