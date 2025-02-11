---
title: PYTR
emoji: 🎵
colorFrom: red
colorTo: indigo
sdk: gradio
sdk_version: 5.15.0
app_file: app.py
pinned: false
license: apache-2.0
---

## on hugging face <a href="https://huggingface.co/spaces/sudo-soldier/PYTR">PYTR</a><br />

**Framework:** Gradio (v5.15.0)  
**License:** Apache 2.0  
  

## Run Locally  

```
git clone https://huggingface.co/spaces/sudo-soldier/PYTR
```

```
cd PYTR
```

## Create and activate Python environment
```
python -m venv env
source env/bin/activate
```

## Install dependencies and run
```
pip install -r requirements.txt
python app.py
```

## Embed the space
```
<iframe 
  src="https://sudo-soldier-pytr.hf.space" 
  frameborder="0" 
  width="850" 
  height="450">
</iframe>
```

## app.py

```

import os
import yt_dlp
import gradio as gr
import re
import subprocess
from pydub import AudioSegment

os.makedirs("downloads", exist_ok=True)

def sanitize_filename(filename):
    """Sanitize filenames by replacing special characters."""
    return re.sub(r'[^a-zA-Z0-9_-]', '_', filename)

def process_youtube_or_audio(url, uploaded_audio, start_time, end_time):
    """Downloads or processes audio, trims it, and exports ringtones."""
    try:
        filename, song_name = None, None

        if url:
            ydl_opts = {
                'format': 'bestaudio/best',
                'outtmpl': 'downloads/%(title)s.%(ext)s',
                'cookiefile': 'cookies.txt'
            }
            with yt_dlp.YoutubeDL(ydl_opts) as ydl:
                info = ydl.extract_info(url, download=True)
                filename = ydl.prepare_filename(info)
                song_name = sanitize_filename(info['title'])
        
        elif uploaded_audio is not None:
            filename = uploaded_audio
            song_name = sanitize_filename(os.path.splitext(os.path.basename(uploaded_audio))[0])
        
        if not filename or not os.path.exists(filename):
            return None, None

        audio = AudioSegment.from_file(filename)
        start_ms, end_ms = int(start_time * 1000), int(end_time * 1000)
        start_ms = max(0, min(start_ms, len(audio)))
        end_ms = max(start_ms, min(end_ms, len(audio)))

        trimmed_audio = audio[start_ms:end_ms]

        mp3_filename = f"downloads/{song_name}.mp3"
        trimmed_audio.export(mp3_filename, format="mp3")

        m4a_filename = f"downloads/{song_name}.m4a"
        result = subprocess.run([
            'ffmpeg', '-y', '-i', mp3_filename, '-vn', '-acodec', 'aac', '-b:a', '192k', m4a_filename
        ], capture_output=True, text=True)

        if result.returncode != 0 or not os.path.exists(m4a_filename):
            print("FFmpeg Error:", result.stderr)
            return mp3_filename, None

        m4r_filename = f"downloads/{song_name}.m4r"
        os.rename(m4a_filename, m4r_filename)

        return mp3_filename, m4r_filename

    except Exception as e:
        print(f"Error: {e}")
        return None, None

# Gradio Interface
with gr.Blocks() as interface:
    gr.HTML("""
        <h1><i class="fas fa-music"></i>&nbsp;PYTR - Python YouTube Ringtones</h1>
        <p>Enter a YouTube URL or upload an audio file to create ringtones.</p>
        <p><a href="https://ringtones.jessejesse.xyz">Ringtones</a></p>
        <p><a href="https://github.com/sudo-self/pytr">GitHub</a></p>
    """)

    with gr.Row():
        youtube_url = gr.Textbox(placeholder="Enter the URL here...", label="YouTube URL")
        uploaded_audio = gr.File(label="Upload Audio File", type="filepath")

    with gr.Row():
        instructions = gr.Textbox(
            value="Android: move the download to the 'Ringtones' folder.\nApple: import download with 'Garage Band' and export to ringtones.",
            label="Install Ringtones",
            interactive=False
        )

    gr.HTML("<h3>Trim Audio</h3>")
    with gr.Row():
        start_time = gr.Slider(0, 20, value=0, label="Start Time (seconds)")
        end_time = gr.Slider(1, 20, value=20, label="End Time (seconds)")
    
    process_button = gr.Button("Create Ringtones")
    
    with gr.Row():
        mp3_download = gr.File(label="Android Ringtone")
        iphone_ringtone = gr.File(label="iPhone Ringtone")
    
    process_button.click(process_youtube_or_audio, inputs=[youtube_url, uploaded_audio, start_time, end_time], outputs=[mp3_download, iphone_ringtone])

interface.launch(share=True)

```

<img width="1440" alt="Screenshot 2025-02-07 at 10 56 21" src="https://github.com/user-attachments/assets/46ac4811-b978-4829-8bcc-81ea184d5ee4" />



