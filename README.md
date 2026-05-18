# N64 Media Split

Desktop tool for Windows that prepares video files for an **N64 media player ROM** on a flash cart (EverDrive, 64Drive, etc.). It splits a normal video file into two matching files on your PC—**H.264 video** and **PCM WAV audio**—that you copy to an SD card alongside your player ROM.

---

## How the N64 media player works

The media player is a **homebrew ROM** you build with [libdragon](https://github.com/DragonMinded/libdragon) (MIPS toolchain + `make`). That ROM runs on real N64 hardware and plays files from the **SD card** on your flash cart. The console does not decode MP4/MKV directly; it reads **pre-converted** media your PC prepared ahead of time.

### What happens on the N64

1. You flash or copy the **player ROM** (for example `sdvideo.z64`) to your cart or SD bootstrap.
2. You put **media files** on the SD card (same folder or a `videos/` folder, depending on your player fork).
3. At runtime the ROM:
   - **Reads** the video stream from storage in chunks (streaming, not loading the whole file into RAM).
   - **Decodes** video on the CPU (or your project’s decoder path) and draws frames to the display.
   - **Plays** the matching audio track in sync (often via libdragon’s audio APIs).
4. The user picks a title from a simple **menu** (SD menu builds) or plays a single embedded clip (EverDrive V2–style embedded ROM builds).

The N64 has very little RAM and a slow CPU compared to a PC, so media must be **small resolution**, **low bitrate**, and in **formats the ROM was written to understand**.

### Two common media formats (don’t mix them up)

| Workflow | Video on SD | Audio on SD | Typical toolchain |
|----------|-------------|-------------|-------------------|
| **Libdragon MPEG-1 player** (official example / portable FMV pack) | `.m1v` (MPEG-1) | `.wav64` (libdragon audio) | `video2n64` GUI + **ffmpeg** + **audioconv64** + **make** |
| **H.264 + WAV player** (this repo’s target) | `.h264` (raw H.264 elementary stream) | `.wav` (16-bit PCM) | **N64 Media Split** + **ffmpeg** only |

This repository is for the **second** row: quick split to `.h264` + `.wav` with the **same base filename** (e.g. `episode01.h264` and `episode01.wav`). Your ROM source must be built to expect those extensions and codecs.

### End-to-end flow (H.264 + WAV)

```
  PC: any video (MP4, MKV, …)
        │
        ▼
  N64 Media Split  ──ffmpeg──►  MyClip.h264  +  MyClip.wav
        │
        ▼
  Copy to SD card next to your player ROM
        │
        ▼
  N64: boot player ROM → select MyClip → playback
```

For the full **MPEG-1 / ROM build** pipeline (menu ROM, embedded ROM, libdragon install), see the separate **N64 Video** portable pack and [libdragon MPEG-1 Player wiki](https://github.com/DragonMinded/libdragon/wiki/MPEG1-Player).

---

## What this converter does

**N64 Media Split** is a small Python app with a simple GUI. For each input video it:

1. **Encodes video** to a raw **H.264** elementary stream (`.h264`):
   - Codec: `libx264`, **baseline** profile, `yuv420p`
   - Scaled (default **320 px wide**, height auto, divisible by 2)
   - Target bitrate default **800k**, fixed GOP for steadier streaming
   - **No audio** in the video file (`-an`)

2. **Extracts audio** to **WAV** (`.wav`):
   - **PCM 16-bit** (`pcm_s16le`)
   - Default **32 kHz**, **mono** (matches common N64 FMV-style settings)
   - Skipped gracefully if the source has no audio track

3. **Names outputs** from the input filename:
   - `Vacation.mp4` → `Vacation.h264` + `Vacation.wav` (spaces → underscores in the stem)

It does **not** build a `.z64` ROM, run `make`, or call `audioconv64`. It only prepares the two files you copy to the SD card.

---

## Requirements

| Component | Required? | Notes |
|-----------|-----------|--------|
| **Python 3** | Yes | Standard library only (`tkinter` for the GUI). |
| **ffmpeg** | Yes | Does all encoding/decoding. [Windows builds](https://www.gyan.dev/ffmpeg/builds/) — add `bin` to PATH or point the GUI at `ffmpeg.exe`. |
| libdragon / MIPS gcc | No | Only needed when **building** your player ROM, not when running this splitter. |

Optional: place `ffmpeg.exe` in `tools\bin\` inside this repo; the app checks that folder automatically.

---

## Quick start (Windows)

1. Install [Python 3](https://www.python.org/downloads/) (check **Add to PATH**).
2. Install **ffmpeg** and ensure `ffmpeg` works in a terminal, **or** browse to `ffmpeg.exe` in the app.
3. Double-click **`Run_N64_Media_Split.bat`**.
4. **Add files** → choose **output folder** → **Convert to .h264 + .wav**.
5. Copy the `.h264` and `.wav` files to your SD card (keep matching names).

### Command line

```powershell
python split_media_cli.py "C:\Videos\clip.mp4" -o "C:\Videos\n64-out"
```

Options: `--scale`, `--bitrate`, `--audio-hz`, `--ffmpeg`.

---

## Default encoding settings

These defaults are a reasonable starting point for N64-style playback; adjust in the GUI if your ROM documentation asks for something different.

| Setting | Default |
|---------|---------|
| Video scale | `320:-2` (320 px wide) |
| Video bitrate | `800k` |
| H.264 profile | `baseline` |
| Pixel format | `yuv420p` |
| Audio | mono, **32000 Hz**, 16-bit PCM |

---

## Project layout

| File | Purpose |
|------|---------|
| `split_media_lib.py` | Conversion logic (ffmpeg commands) |
| `split_media_gui.py` | Tkinter GUI |
| `split_media_cli.py` | Command-line interface |
| `Run_N64_Media_Split.bat` | Double-click launcher |

---

## Troubleshooting

- **“ffmpeg was not found”** — Install ffmpeg, add it to PATH, set the `FFMPEG` env var, or use **Browse…** in the GUI.
- **Video plays but no sound** — Source may have no audio; only `.h264` is produced.
- **ROM won’t play files** — Confirm your player ROM expects **`.h264` + `.wav`**, not `.m1v` + `.wav64`. Rebuild or use the correct converter for your ROM.

---

## License

Add your license here if you publish the repo (e.g. MIT).
