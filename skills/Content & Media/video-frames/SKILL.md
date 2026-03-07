---
id: video-frames
category: Content & Media
author: openclaw
name: Video Frames
description: Extract frames or short clips from videos using ffmpeg.
requires: [".mp4", ".mov", ".mkv", ".webm"]
examples:
  - "Extract a frame at 00:00:10 from video.mp4 and save it as frame-10s.jpg."
  - "Generate 12 evenly spaced thumbnail frames from this video and save them to ./thumbs/."
---

# Video Frames (ffmpeg)

Extract a single frame from a video, or create quick thumbnails for inspection.

## Quick start

First frame:

```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --out /tmp/frame.jpg
```

At a timestamp:

```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --time 00:00:10 --out /tmp/frame-10s.jpg
```

## Notes

- Prefer `--time` for “what is happening around here?”.
- Use a `.jpg` for quick share; use `.png` for crisp UI frames.
