---
id: 7w56jr8akg
title: "R4 video: vellum render --video, Render Animation (Ctrl+F12)"
order: "b1S"
created: 2026-11-03T21:04:12Z
updated: 2026-11-03T21:04:12Z
version: 1
Status: "Done"
Milestone: "Release 0.1"
Area: "Platform"
Priority: "Medium"
Due: 2026-11-03
Notes: "2026-11-03. vellum::video::VideoWriter pipes raw RGBA to ffmpeg (or VELLUM_FFMPEG) at 24 fps; mp4/mov/mkv H.264 (libx264 -tune animation, else OpenH264, else MPEG-4), webm VP9/VP8, gif with palettegen; odd sizes padded; a missing ffmpeg is reported before rendering. --out x.mp4 works too; --video with --out <dir> writes layers as well. Preview: render_job.rs thread with its own GPU device and unsaved overrides, video or PNG frames, progress bar, Cancel. Tests for codecs, a real encode, and the job. Branch r4-video, not committed. Replaces the 'R4 video output' row."
---
