<div align="center">

# Hybrid Computer Vision System for Automated Industrial Pallet Inspection

*Combining neural detection, spatial inference, and rule-based validation*
*for sub-5-second, near-zero-error part counting*

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![YOLOv8](https://img.shields.io/badge/YOLOv8-Ultralytics-00BFFF?style=flat-square)](https://github.com/ultralytics/ultralytics)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.x-5C3EE8?style=flat-square&logo=opencv&logoColor=white)](https://opencv.org)
[![PyQt5](https://img.shields.io/badge/GUI-PyQt5-41CD52?style=flat-square)](https://riverbankcomputing.com/software/pyqt/)
[![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-0078D4?style=flat-square&logo=windows&logoColor=white)](https://microsoft.com)

</div>

---

## Abstract

Manual part counting on industrial pallets is a high-frequency, error-prone task that limits quality control throughput in manufacturing environments. This project describes the design and deployment of a production-grade hybrid vision system that reduces per-pallet inspection time from approximately **two minutes to under five seconds** while eliminating operator counting errors. The system integrates a custom-trained YOLOv8 detection model with a spatial grid-inference layer and an OpenCV-based validation pass, operating in real time on a live industrial camera feed. The system has been deployed in a production factory setting across three pallet configurations (100-, 500-, and 1000-slot modes), demonstrating robust performance under industrial lighting variation, partial occlusion, and part reflectivity.

> **Keywords:** YOLOv8 · object detection · industrial inspection · computer vision · Hough transform · PyQt5 · manufacturing automation

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [System Overview](#2-system-overview)
3. [Detection Pipeline](#3-detection-pipeline)
4. [Pallet Configuration Modes](#4-pallet-configuration-modes)
5. [Results](#5-results)
6. [Production Engineering Features](#6-production-engineering-features)
7. [Software Architecture](#7-software-architecture)
8. [Discussion](#8-discussion)
9. [Getting Started](#9-getting-started)
10. [Future Work](#10-future-work)

---

## 1. Problem Statement

In high-volume manufacturing, pallet inspection is performed at the end of each production run to verify part count before shipment or downstream assembly. For trays carrying 500 or 1,000 small components, manual counting is cognitively demanding and time-consuming.

| Pain Point | Impact |
|---|---|
| ~2 minutes per pallet | Shift-level throughput bottleneck |
| Fatigue-induced miscounts | Variable quality, rework cost |
| Manual paper logging | Transcription errors, audit risk |
| Operator-dependent pace | Non-scalable throughput ceiling |

---

## 2. System Overview

SYSTEM is a desktop application for Windows 10/11 built in Python 3.10. A Hikrobot GigE / USB3 Vision industrial camera provides the frame source. The operator workflow is minimal by design:

```
1. Select pallet mode
2. Press COUNT  →  < 5 seconds
3. Press SAVE   →  Excel log + screenshot generated
```

### End-to-End Pipeline

```
┌─────────────────┐
│  Camera Feed    │  Hikrobot GigE / USB3 Vision
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Stage 1        │  YOLOv8 — neural slot detection
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Stage 2        │  Grid Gap Inference — spatial reconstruction
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Stage 3        │  OpenCV Validation — Hough + blob cross-check
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Output         │  Auto count · Judgement · Excel log · Screenshot
└─────────────────┘
```

---

## 3. Detection Pipeline

Production environments are messy — lighting changes, parts reflect, slots get occluded. Pure neural inference is insufficient. SYSTEM uses a **three-stage hybrid pipeline** combining AI with rule-based spatial reasoning.

### Stage 1 — YOLOv8 Detection

A custom YOLOv8n model trained on labelled pallet tray images. Detects two classes: filled slot and empty slot. Confidence and IoU thresholds are tuned independently per pallet mode.

```python
results = model(frame, conf=0.25, iou=0.45)
```

**Features:**
- Real-time inference on live camera frames
- Configurable confidence / IoU thresholds per mode
- Separate model tuning per pallet configuration

---

### Stage 2 — Spatial Grid-Gap Inference

When YOLO misses a detection, the absence of an expected box at a known grid coordinate is itself a signal. This stage reconstructs the full tray grid from detected boxes and infers missing cells analytically.

```
Detected slots → Row clustering → Grid reconstruction
              → Missing-cell inference → False-positive suppression
```

- Missing inferred slots rendered as **yellow overlays** in the GUI
- Exploits fixed tray geometry as a structural prior
- Suppresses false positives from surface texture and glare

---

### Stage 3 — OpenCV Validation Layer

For dense 100-slot pallets, an additional pass runs Hough Circle Transform and blob detection, cross-validating results against Stage 1 via IoU comparison. Catches small missed parts; suppresses duplicates.

**Methods used:**
- `cv2.HoughCircles` — circular part detection
- Blob detection — low-contrast part recovery  
- IoU cross-validation — duplicate suppression with YOLO boxes

---

## 4. Pallet Configuration Modes

| Mode | Slots | Detection Strategy | Count Formula |
|------|-------|--------------------|---------------|
| A | 100 | Detect filled parts | `count = detections` |
| B | 500 | Detect empty blanks | `count = 500 − blanks` |
| C | 1,000 | Detect empty blanks | `count = 1000 − blanks` |

Switching modes at runtime resets grid state, clears detection history, reloads thresholds, and reconfigures overlays — **no restart required**.

```python
# Dynamic formula switching
if mode == "100":
    parts = filled_detections
elif mode == "500":
    parts = 500 - blank_detections
elif mode == "1000":
    parts = 1000 - blank_detections
```

---

## 5. Results

Measured over 50 consecutive pallets per condition. Error rate determined by ground-truth comparison with manual recount.

| Metric | Before (Manual) | After (SYSTEM) | Change |
|--------|----------------|----------------|--------|
| Inspection time | ~2 min | **< 5 sec** | ~24× faster |
| Counting error rate | Variable (fatigue) | **Near-zero** | Effectively eliminated |
| Record keeping | Manual paper log | **Auto Excel + screenshot** | Fully automated |
| Throughput ceiling | Operator-limited | **Camera-limited** | Bottleneck removed |
| Repeatability | Operator-dependent | **Deterministic** | Standardised |

---

## 6. Production Engineering Features

### Power Suspend Recovery

Factory PCs sleep. Cameras disconnect. Windows resumes unpredictably. SYSTEM handles all of it.

- Win32 `WM_POWERBROADCAST` listener on a background thread
- Graceful camera shutdown on suspend
- Automatic reconnect on resume — zero operator intervention

### Live Excel Hot Reload

Operators can update part codes and operator lists during a running shift, without restarting.

```
Excel file modified
        ↓
QFileSystemWatcher detects change (100ms debounce)
        ↓
Database reloads automatically in running app
```

### Atomic Undo Stack

Every COUNT operation pushes a full application state snapshot. Undo restores atomically:

```python
history.push({
    "part_qty": ...,
    "overall_qty": ...,
    "pallet_totals": ...,
    "lcd_state": ...,
    "judgement": ...
})
```

### Operator-Friendly Interface

Designed for real factory floor constraints — gloves, fatigue, time pressure.

- Touchscreen and on-screen keyboard support
- Smart autocomplete for part codes and operator names
- Oversized buttons for gloved use
- Colour-coded LCD-style counters visible from a distance
- Fullscreen operation

---

## 7. Software Architecture

### Project Structure

```
pallet_system/
│
├── main.py                    # Bootstrap + model loading
│
├── code/
│   ├── GUI.py                 # Operator interface
│   ├── camera_wrapper.py      # MVS SDK · init · recovery
│   ├── camera_worker.py       # Frame grab · preprocessing · QThread
│   ├── yolo_infer.py          # 3-stage detection pipeline
│   ├── actions.py             # COUNT · SAVE · UNDO · RESET
│   └── exceptions.py
│
├── utils/
│   ├── file_utils.py
│   └── keyboard_utils.py
│
├── SetRecord/                 # Runtime Excel databases
└── runs/                      # YOLO training outputs
```

### Runtime Data Flow

```
Hikrobot Camera
      │
      ▼
camera_wrapper.py ── MVS SDK · stream · reconnect · power recovery
      │
      ▼
camera_worker.py ─── frame grab · preprocessing · QThread dispatch
      │
      ▼
yolo_infer.py ─────── Stage 1 (YOLO) → Stage 2 (Grid) → Stage 3 (CV)
      │
      ▼
GUI.py ──────────────  display · counters · mode · operator controls
      │
      ▼
actions.py ──────────  COUNT · SAVE · UNDO · RESET
      │
      ▼
Data layer ──────────  Excel log · screenshot · QFileSystemWatcher
```

### Technology Stack

| Component | Technology |
|-----------|------------|
| Language | Python 3.10+ |
| AI Detection | YOLOv8 (Ultralytics) |
| Computer Vision | OpenCV |
| GUI Framework | PyQt5 |
| Camera SDK | Hikrobot MVS |
| Data I/O | pandas + openpyxl |
| Deployment | PyInstaller |

---

## 8. Discussion

### Hybrid Pipelines in Structured Environments

A key finding is that neural detection alone is insufficient for production reliability. YOLOv8 achieves high accuracy on clean, well-lit images but degrades under glare, shadows, and occlusion. Because tray geometry is fixed and known, missing detections can be partially resolved analytically — if a grid position should contain a detection and none is found, spatial inference fills the gap with high confidence. This hybrid strategy consistently outperformed threshold tuning alone.

> *In domains with strong structural priors, the combination of learned perception with deterministic spatial reasoning outperforms either approach in isolation.*

### Hardware Integration Complexity

Industrial cameras introduce failure modes absent from standard USB cameras: GigE packet loss, USB3 bandwidth contention, and power management disconnects that are not always propagated cleanly to application callbacks. Building explicit recovery logic was as important to system reliability as ML accuracy.

### Operator-Centred Design

The most-valued features — undo, large buttons, autocomplete — were not in the initial specification. They emerged from observing operators on the first prototype under real shift conditions. Factory-floor software must be validated with its actual users, whose constraints differ qualitatively from standard office software users.

---

## 9. Getting Started

### Requirements

- Windows 10/11
- Hikrobot GigE / USB3 camera + MVS SDK
- Python 3.10+
- CUDA-capable GPU (recommended for real-time inference)

### Installation

```bash
git clone https://github.com/yourusername/pallet-inspection-system
cd pallet-inspection-system
pip install -r requirements.txt
```

### Model Training

Weights are excluded from this repository. Train your own:

```bash
yolo train \
    model=yolov8n.pt \
    data=your_data.yaml \
    epochs=200
```

Place trained weights at:

```
runs/train100_200_epochs/weights/best.pt
```

### Output Structure

```
SetRecord/
└── 2025/
    └── May/
        └── RESULT.xlsx          # Date · pallet type · lot · part code
                                 # operator · qty · judgement

PhotoRecord/
└── 2025/
    └── May/
        └── 2025-05-14/
            └── PARTCODE_LOT_TIMESTAMP.jpg
```

---

## 10. Future Work

- Multi-camera synchronisation for simultaneous adjacent-pallet inspection
- Centralised production dashboard aggregating results across workstations
- Statistical defect analytics and shift-level trend monitoring
- Active-learning retraining pipeline using operator-confirmed corrections
- Remote monitoring interface for supervisors
- Real-time shortage alert notifications

---

<div align="center">

*Industrial AI built for the real world.*

**[LinkedIn](https://my.linkedin.com/in/qaisara-mardhiah)** · **[Portfolio](https://qaisaram.github.io/PALLET-INSPECTION-SYSTEM/)**

</div>
