# YouTube Video Download, Merge & Smooth Trim Documentation

## Objective
This documentation explains the workflow used to:
- Download a YouTube video in high quality
- Merge separate audio and video streams
- Trim a specific portion of the video smoothly using FFmpeg

---

# 1. Installing yt-dlp

## Purpose
`yt-dlp` is used to download YouTube videos in high quality.

## Command

```bash
pip install yt-dlp
```

---

# 2. Downloading High Quality YouTube Video

## Command

```bash
yt-dlp -f bestvideo+bestaudio "https://www.youtube.com/watch?v=G-5Kt4jBT-4"
```

## Explanation

- `-f bestvideo+bestaudio`
  - Downloads best available video quality
  - Downloads best available audio quality

Because YouTube stores audio and video separately, two files were downloaded.

### Downloaded Video File

```text
.f399.mp4
```

### Downloaded Audio File

```text
.f251.webm
```

---

# 3. Installing FFmpeg

## Purpose
FFmpeg is used for:
- Merging audio and video
- Trimming videos
- Video processing

## Installation Command

```bash
winget install ffmpeg
```

## Verification

```bash
ffmpeg -version
```

---

# 4. Merging Audio and Video

## Purpose
To combine:
- Video stream (`.mp4`)
- Audio stream (`.webm`)

into a single playable MP4 file.

## Command

```bash
ffmpeg -i "Hon'ble CM of AP Sri Nara Chandrababu Naidu Participates in the “Matsyakara Sevalo” Programme [G-5Kt4jBT-4].f399.mp4" -i "Hon'ble CM of AP Sri Nara Chandrababu Naidu Participates in the “Matsyakara Sevalo” Programme [G-5Kt4jBT-4].f251.webm" -c:v copy -c:a aac final_video.mp4
```

## Output

```text
final_video.mp4
```

## Explanation

- `-c:v copy`
  - Copies video without re-encoding
  - Faster processing

- `-c:a aac`
  - Converts audio into AAC format

---

# 5. Trimming a Specific Video Portion

## Objective

Trim video from:

```text
2:01:52 → 2:03:46
```

---

# 6. Initial Trim Method (Fast)

## Command

```bash
ffmpeg -ss 02:01:52 -to 02:03:46 -i "final_video.mp4" -c copy trimmed_clip.mp4
```

## Problem Observed

- Audio started immediately
- Video froze for 1–2 seconds

## Reason

Using:

```bash
-c copy
```

causes trimming at nearest keyframe instead of exact frame.

---

# 7. Smooth Professional Trim (Recommended)

## Command

```bash
ffmpeg -ss 02:01:52 -to 02:03:46 -i "final_video.mp4" -c:v libx264 -c:a aac smooth_trim.mp4
```

## Output

```text
smooth_trim.mp4
```

## Why This Works Better

- Re-encodes video properly
- Eliminates freeze/stuck frame issue
- Produces smooth playback

## Parameters

- `-c:v libx264`
  - Re-encodes video using H.264 codec

- `-c:a aac`
  - Encodes audio in AAC format

---

# Final Workflow Summary

1. Install yt-dlp
2. Download YouTube video
3. Install FFmpeg
4. Merge video + audio
5. Trim required segment
6. Fix trim freeze issue using smooth re-encoding

---

# Useful Commands Summary

## Install yt-dlp

```bash
pip install yt-dlp
```

## Download Best Quality Video

```bash
yt-dlp -f bestvideo+bestaudio "YOUTUBE_URL"
```

## Install FFmpeg

```bash
winget install ffmpeg
```

## Merge Audio + Video

```bash
ffmpeg -i video.mp4 -i audio.webm -c:v copy -c:a aac output.mp4
```

## Fast Trim

```bash
ffmpeg -ss START -to END -i input.mp4 -c copy output.mp4
```

## Smooth Professional Trim

```bash
ffmpeg -ss START -to END -i input.mp4 -c:v libx264 -c:a aac output.mp4
```

---

# Final Output Files

| File Name | Purpose |
|---|---|
| final_video.mp4 | Merged video + audio |
| smooth_trim.mp4 | Smooth trimmed clip |

---

# Conclusion

This workflow demonstrates a professional video-processing pipeline using:
- yt-dlp
- FFmpeg

for:
- High-quality YouTube video downloading
- Audio/video merging
- Smooth video trimming without frame freezing issues.