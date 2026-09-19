# H.264 GOP Structure & Bitrate Test Vectors
[![Basis Video](https://img.shields.io/badge/Hosted%20by-Basis%20Video-blue)](https://basisvideo.com)

### Test bitstreams and reproducible FFmpeg encoding recipes
A reference suite and FFmpeg commands for generating isolated Group of Pictures (GOP) test streams. Built as reproducible baselines for video engineers and streaming developers to stress-test hardware/software decoders, inspect display vs. decode order, validate ABR rendition keyframe alignment, and debug error recovery.


## Test Stream Catalog & Downloads

| # | Test Clip | Key Isolated Variable | Specs | Direct Download |
|---|---|---|---|---|
| **01** | `gop_01_allintra.mp4` | All-Intra (I-frame only) | 720p @ 25 fps | [Download MP4](https://streams.basisvideo.com/gop_01_allintra.mp4) |
| **02** | `gop_02_ippp_closed.mp4` | IPPP, Closed GOP | 720p @ 25 fps | [Download MP4](https://streams.basisvideo.com/gop_02_ippp_closed.mp4) |
| **03** | `gop_03_ibbp_closed.mp4` | IBBP, Closed GOP | 720p @ 25 fps | [Download MP4](https://streams.basisvideo.com/gop_03_ibbp_closed.mp4) |
| **04** | `gop_04_open.mp4` | Open GOP | 720p @ 25 fps | [Download MP4](https://streams.basisvideo.com/gop_04_open.mp4) |
| **05** | `gop_05_intra_refresh.mp4` | Periodic Intra Refresh | 720p @ 25 fps | [Download MP4](https://streams.basisvideo.com/gop_05_intra_refresh.mp4) |
| **06a** | `gop_06a_short.mp4` | Short GOP (0.5s / 30 frames) | 720p @ 60 fps | [Download MP4](https://streams.basisvideo.com/gop_06a_short.mp4) |
| **06b** | `gop_06b_long.mp4` | Long GOP (4.0s / 240 frames) | 720p @ 60 fps | [Download MP4](https://streams.basisvideo.com/gop_06b_long.mp4) |
| **07** | `gop_07_scenecut.mp4` | Scene-Cut Adaptive GOP | 720p @ 25 fps | [Download MP4](https://streams.basisvideo.com/gop_07_scenecut.mp4) |
| **08** | `gop_08_corrupted.h264` | Truncated Mid-GOP (Annex-B) | 720p @ 25 fps | [Download .h264](https://streams.basisvideo.com/gop_08_corrupted.h264) |
| **09a** | `gop_09a_cbr.mp4` | Hard CBR (VBV constrained) | 720p @ 25 fps | [Download MP4](https://streams.basisvideo.com/gop_09a_cbr.mp4) |
| **09b** | `gop_09b_vbr.mp4` | VBR (CRF-based) | 720p @ 25 fps | [Download MP4](https://streams.basisvideo.com/gop_09b_vbr.mp4) |

> 📊 **In-Depth Metrics:**  
> Bitstream metrics and detailed structural data are available at **[Basis Video: GOP Structure Test Streams](https://basisvideo.com/video-encoding/free-h-264-test-streams-ffmpeg-recipes-gop-bitrate)**.

---

## Generation Recipes (FFmpeg & libx264)

- **Encoding Pipeline:** Generated from open reference masters using FFmpeg (libx264).
- **Frame Rates:** 25 fps baseline (Clips #6a and #6b use the 60 fps source).
- **Requirements:** A standard installation of FFmpeg with libx264 enabled.

### 1. All-Intra (Zero Temporal Dependency)

```bash
ffmpeg -i input.mp4 -c:v libx264 -g 1 -bf 0 -an gop_01_allintra.mp4
```

### 2. IPPP only, closed GOP
```bash
ffmpeg -i input.mp4 -c:v libx264 -g 50 -bf 0 \
  -x264-params "min-keyint=50:no-open-gop=1:scenecut=0" \
  -an gop_02_ippp_closed.mp4
```

### 3. IBBP closed GOP
```bash
ffmpeg -i input.mp4 -c:v libx264 -g 50 -bf 2 \
  -x264-params "min-keyint=50:no-open-gop=1:scenecut=0" \
  -an gop_03_ibbp_closed.mp4
```

### 4. Open GOP — Cross-GOP referencing
```bash
ffmpeg -i input.mp4 -c:v libx264 -g 50 -bf 3 \
  -x264-params "min-keyint=50:open-gop=1:scenecut=0" \
  -an gop_04_open.mp4
```

### 5. Non-IDR, Periodic Intra-Refresh
```bash
ffmpeg -i input.mp4 -c:v libx264 -bf 0 \
  -x264-params "intra-refresh=1:keyint=50" \
  -an gop_05_intra_refresh.mp4
```

### 6. GOP Length impact on bitrate 

Relation of compression-efficiency with GOP Length.

#### a. Short GOP

```bash
ffmpeg -i input.mp4 \
  -c:v libx264 \
  -qp 26 \
  -preset slow \
  -g 30 -keyint_min 30 -forced-idr 1 \
  -x264-params "scenecut=0:aq-mode=0:mbtree=0:ipratio=1.0:pbratio=1.0:qpstep=0:chroma-qp-offset=0" \
  -c:a copy \
  gop_06a_short.mp4
```
  
### b. Long GOP

```bash
ffmpeg -i input.mp4 \
  -c:v libx264 \
  -qp 26 \
  -preset slow \
  -g 240 -keyint_min 240 -forced-idr 1 \
  -x264-params "scenecut=0:aq-mode=0:mbtree=0:ipratio=1.0:pbratio=1.0:qpstep=0:chroma-qp-offset=0" \
  -c:a copy \
  gop_06b_long.mp4
```

# 7. Scene-cut adaptive GOP

```bash
ffmpeg -i input.mp4 -c:v libx264 -g 250 -bf 2 \
  -x264-params "min-keyint=25:scenecut=40" \
  -an gop_07_scenecut.mp4
```

# 8. Corrupted / truncated elementary stream
Raw Annex-B stream without an MP4 container.

```bash
ffmpeg -i gop_03_ibbp_closed.mp4 -c copy -bsf:v h264_mp4toannexb \
  -f h264 gop_03_ibbp_closed.h264

SIZE=$(stat -c%s gop_03_ibbp_closed.h264)

head -c $((SIZE * 60 / 100)) gop_03_ibbp_closed.h264 > gop_08_corrupted.h264
```

Inspect or play it via FFplay or a bitstream analyzer:

```bash
ffplay -f h264 gop_08_corrupted.h264
```

### 9 CBR vs VBR

Both clips share the exact GOP structure — same keyframe interval, same B-frame count, only the rate-control mode differs.

#### a. CBR


```bash
ffmpeg -i input.mp4 -c:v libx264 -g 50 -bf 2 \
  -b:v 4M -minrate 4M -maxrate 4M -bufsize 4M \
  -x264-params "min-keyint=50:no-open-gop=1:scenecut=0:nal-hrd=cbr" \
  -an gop_09a_cbr.mp4
```

#### b. VBR

```bash
ffmpeg -i input.mp4 -c:v libx264 -g 50 -bf 2 -crf 20 \
  -x264-params "min-keyint=50:no-open-gop=1:scenecut=0" \
  -an gop_09b_vbr.mp4
```

---


**Repository License**

Licensed under the MIT License. Reference media files are open for public testing, research, and pipeline validation.

---

### Hosted & Maintained by

This test suite and documentation are maintained by **[Basis Video](https://basisvideo.com)**.
