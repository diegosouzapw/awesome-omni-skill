---
name: cv181x-media
description: "Expert guide for CV181X/CV182X/CV180X (SG200X) multimedia development using CVI MPI API. Use this skill when working with: VI (video input/camera/ISP), VPSS (video processing/scaling/crop), VENC (H.264/H.265/JPEG encoding), VDEC (decoding), VB (video buffer pools), SYS binding, or any CVI_* API calls. Covers camera pipeline setup, offline VPSS processing, VB pool planning, and error diagnosis (ERR_VPSS_NOBUF, ERR_VB_NOBUF). API details in references/."
version: 2.3.0
---

# CV181X/CV182X/CV180X Multimedia Skill

## Complete Reference Index

All 18 reference files with searchable keywords:

| File | Keywords | Key APIs |
|------|----------|----------|
| `references/vi.md` | camera, sensor, ISP, MIPI, DEV, PIPE, CHN, video input | `CVI_VI_*`, `CVI_ISP_*`, `CVI_MIPI_*` |
| `references/vpss.md` | scaling, crop, resize, rotate, cvtColor, format conversion, group, channel | `CVI_VPSS_*`, `SendFrame`, `GetChnFrame` |
| `references/venc.md` | encode, H.264, H.265, HEVC, JPEG, MJPEG, bitrate, GOP | `CVI_VENC_*`, `SendFrame`, `GetStream` |
| `references/vdec.md` | decode, JPEG decode, H.264 decode, bitstream | `CVI_VDEC_*`, `SendStream`, `GetFrame` |
| `references/vb.md` | buffer pool, VB, memory, block, pool sizing, CommonPool | `CVI_VB_*`, `GetBlock`, `ReleaseBlock` |
| `references/sys.md` | bind, unbind, init, exit, VI_VPSS_MODE, VPSS_MODE | `CVI_SYS_*`, `Bind`, `SetVIVPSSMode` |
| `references/vo.md` | display, output, LCD, HDMI, layer, framebuffer | `CVI_VO_*`, `SendFrame`, `ChnShow` |
| `references/rgn.md` | OSD, overlay, region, text, graphics, privacy mask | `CVI_RGN_*`, `AttachToChn`, `SetBitMap` |
| `references/gdc.md` | distortion, fisheye, LDC, rotation, mesh, warp | `CVI_GDC_*`, `AddCorrectionTask` |
| `references/audio.md` | audio, microphone, speaker, AI, AO, AENC, ADEC, VQE | `CVI_AI_*`, `CVI_AO_*`, `CVI_AENC_*` |
| `references/ion.md` | ION, DMA, cache, flush, invalidate, physical address | `CVI_SYS_IonAlloc`, `IonFlushCache` |
| `references/debug.md` | /proc, log, debug, status, diagnostics | `/proc/cvitek/*`, log level |
| `references/troubleshooting.md` | error, ERR_VPSS_NOBUF, ERR_VB_NOBUF, failure, fix | error codes, solutions |
| `references/binding-cookbook.md` | concurrent, dual pipeline, online, offline, scenario | multi-scenario design |
| `references/integration-guide.md` | integration, cross-module, ownership, lifecycle | system design rules |
| `references/platform.md` | SDK, path, pixel format, CV181X, CV182X, CV180X | platform differences |
| `references/overview.md` | architecture, workflow, pipeline, introduction | system overview |
| `references/scenarios.md` | example, surveillance, doorbell, NVR, application | complete examples |

---

## Quick Lookup

### By Task

| Task | Reference |
|------|-----------|
| Camera capture | `references/vi.md` |
| Video scaling/crop/rotate | `references/vpss.md` |
| H.264/H.265/JPEG encode | `references/venc.md` |
| JPEG/H.264 decode | `references/vdec.md` |
| Buffer pool planning | `references/vb.md` |
| Module binding | `references/sys.md` |
| Video display | `references/vo.md` |
| OSD/overlay | `references/rgn.md` |
| Fisheye correction | `references/gdc.md` |
| Audio capture/playback | `references/audio.md` |
| DMA memory | `references/ion.md` |
| Debugging | `references/debug.md` |
| Error diagnosis | `references/troubleshooting.md` |

### By Error Code

| Error | Reference |
|-------|-----------|
| `ERR_VPSS_NOBUF (0xc006800e)` | `references/troubleshooting.md` |
| `ERR_VB_NOBUF` | `references/vb.md` |
| `ERR_VENC_NOBUF` | `references/venc.md` |
| `ERR_VI_NOTREADY` | `references/vi.md` |
| Bind failure | `references/sys.md` |

### By Pipeline

| Pipeline | References |
|----------|------------|
| Camera -> VPSS (online) | `vi.md`, `vpss.md`, `sys.md` |
| File -> VPSS (offline) | `vpss.md`, `vb.md` |
| JPEG decode -> VPSS | `vdec.md`, `vpss.md` |
| VPSS -> VENC | `venc.md`, `sys.md` |
| Dual pipeline | `references/binding-cookbook.md` |

---

## Private Operating Procedures

### 1. Authoritative Source Order
1. SDK headers in `cvi_mpi/include/`
2. SDK samples in `cvi_mpi/sample/`
3. Never: external repos without header verification

### 2. VPSS Input Ownership
- VPSS group for ONE input type: ISP or MEM
- Never switch between Bind and SendFrame at runtime
- Concurrent ISP + MEM: separate groups and VB pools

### 3. VI-VPSS Mode Selection

| Mode | Use Case |
|------|----------|
| `VI_OFFLINE_VPSS_ONLINE` | Camera pipeline (recommended) |
| `VI_OFFLINE_VPSS_OFFLINE` | Maximum flexibility |
| `VI_ONLINE_VPSS_ONLINE` | Lowest latency |

### 4. Binding Requirements
- Control plane: `CVI_SYS_Bind` establishes relationship
- Data plane: Mode setting determines data flow
- Binding required even in ISP direct mode
- Unbind before stop; bind after start

### 5. Camera Readiness
- Warm up pipeline after start
- Discard first few frames before capture

### 6. VENC SendFrame Memory
- Must use VB pool, not raw ION
- Use `CVI_VB_GetBlock()`, set `frame.u32PoolId`

### 7. Diagnostics
On failure: `/proc/cvitek/{sys,vb,vi,vpss,vdec,venc}`

---

## Module Init Order

```text
1. CVI_VB_SetConfig + CVI_VB_Init
2. CVI_SYS_Init
3. CVI_SYS_SetVIVPSSMode
4. CVI_SYS_SetVPSSModeEx
5. VI: sensor -> mipi -> dev -> pipe -> isp -> chn
6. VPSS: CreateGrp -> SetChnAttr -> EnableChn -> StartGrp
7. CVI_SYS_Bind
8. Warmup
```

---

## VPSS Mode Reference

### VPSS_MODE_DUAL
```text
DEV0 = VPSS_INPUT_MEM   -> Offline (SendFrame)
DEV1 = VPSS_INPUT_ISP   -> Camera (Bind)
```

### VpssDev Selection
| Input | VpssDev |
|-------|---------|
| Camera (ISP) | 1 |
| File/Memory | 0 |

---

## VB Pool Sizing

```text
Size = COMMON_GetPicBufferSize(w, h, fmt, DATA_BITWIDTH_8, COMPRESS_MODE_NONE, 0)

Counts: Camera=4-6, VPSS=3-4, Encode=2-3, Decode=4-6
```

---

## Debug Commands

```bash
cat /proc/cvitek/sys | grep -A 10 "BIND RELATION"
cat /proc/cvitek/vb | grep -A 20 "PoolId"
cat /proc/cvitek/vi | grep -A 5 "VI CHN STATUS"
cat /proc/cvitek/vpss | grep -A 20 "WORK STATUS"
echo "VI=7" > /proc/cvitek/log
```

---

**Version**: 2.4.0 | **Updated**: 2026-01-20 | **Platform**: SG200X
