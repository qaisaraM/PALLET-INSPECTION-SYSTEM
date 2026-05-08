🔗 [View Source Code](https://github.com/qaisaraM/PALLET-INSPECTION-SYSTEM)  
🌐 [Live Demo](https://qaisaram.github.io/PALLET-INSPECTION-SYSTEM/)

<div align="center">

# ⚡ PALLET INSPECTION SYSTEM

## AI-Powered Industrial Pallet Inspection System

### *From 2-minute manual counting → to 5-second AI inspection.*

<img src="https://img.shields.io/badge/STATUS-LIVE%20ON%20FACTORY%20FLOOR-00C896?style=for-the-badge&logo=vercel&logoColor=white"/>
<img src="https://img.shields.io/badge/YOLOv8-Industrial%20Vision-FF6B35?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
<img src="https://img.shields.io/badge/OpenCV-Real--Time%20CV-5C3EE8?style=for-the-badge&logo=opencv"/>
<img src="https://img.shields.io/badge/PyQt5-Touchscreen%20GUI-41CD52?style=for-the-badge"/>

---

### 🏭 Built for real manufacturing operators

### 🤖 Powered by computer vision + spatial inference

### ⚡ Deployed in production on an active factory line

</div>

---

# 🎯 The Problem

Imagine counting **500 tiny parts** on every pallet.
Now repeat that for an entire shift.

Operators were spending:

* ⏱ ~2 minutes per pallet
* 😵 Fighting fatigue-induced counting mistakes
* 📋 Writing logs manually
* 🚧 Slowing down QC throughput

In manufacturing, even small delays multiply fast.

---

# 🚀 The Solution

SYSTEM transforms pallet inspection into a **single-button AI workflow**.

```text
📷 Camera Feed
      ↓
🧠 YOLOv8 Detection
      ↓
📐 Grid Gap Inference
      ↓
🔍 Secondary CV Validation
      ↓
📊 Auto Count + Excel Report
```

### Result?

✅ Accurate counts
✅ Automatic judgement
✅ Screenshot evidence
✅ Excel logging
✅ < 5 second inspection cycle

---

# 🎥 Demo

> 📸 Add a GIF here showing:
>
> * Live camera feed
> * YOLO boxes rendering
> * Counters updating
> * Final OK / SHORTAGE judgement
> * Save-to-Excel flow

---

# 📈 Impact

<div align="center">

| 🔧 Metric         | Before           | After            |
| ----------------- | ---------------- | ---------------- |
| ⏱ Inspection Time | ~2 minutes       | **< 5 seconds**  |
| ❌ Human Error     | Variable         | **Near-zero**    |
| 📋 Record Keeping | Manual           | **Automatic**    |
| 🏭 Throughput     | Operator-limited | **Scalable**     |
| 🔁 Repeatability  | Inconsistent     | **Standardized** |

</div>

---

# 🧠 Detection Pipeline

## 🔬 Multi-Layer Vision System

Production environments are messy.

Lighting changes.
Parts reflect light.
Slots get occluded.
Pure YOLO isn't enough.

So SYSTEM combines **AI + rule-based spatial reasoning**.

---

## 🥇 Stage 1 — YOLOv8 Detection

Custom-trained YOLOv8 model trained on pallet trays.

### Features

* 🎯 Detects filled or empty slots
* ⚙ Configurable confidence / IOU thresholds
* 🧩 Separate tuning per pallet mode
* ⚡ Real-time inference

```python
results = model(frame, conf=0.25, iou=0.45)
```

---

## 🥈 Stage 2 — Grid Gap Inference

When YOLO misses detections:

✅ Spatial clustering reconstructs the pallet grid
✅ Missing cells are inferred
✅ Empty regions validated against tray geometry
✅ False positives suppressed intelligently

### Why this matters

Industrial trays are highly structured.

That means geometry itself becomes a powerful signal.

```text
Detected slots → Row clustering → Grid expansion → Missing-cell inference
```

🟨 Missing slots render as yellow overlays in the UI.

---

## 🥉 Stage 3 — OpenCV Validation Layer

For dense 100-slot pallets:

* 🔵 Hough Circle Transform
* ⚪ Blob detection
* 🔍 IOU cross-validation with YOLO
* 🚫 Duplicate suppression

This catches tiny missed parts that neural detection occasionally skips.

---

# 🏭 Pallet Modes

<div align="center">

| Mode           | Slots | Detection Strategy  |
| -------------- | ----- | ------------------- |
| 🔹 100-Pallet  | 100   | Detect filled parts |
| 🔸 500-Pallet  | 500   | Detect empty blanks |
| 🔺 1000-Pallet | 1000  | Detect empty blanks |

</div>

### Dynamic Formula Switching

```python
100-pallet  → parts = detections
500-pallet  → parts = 500 - blanks
1000-pallet → parts = 1000 - blanks
```

Changing modes automatically:

* Resets grid state
* Clears detection history
* Updates UI counters
* Reloads thresholds
* Reconfigures overlays

⚡ No restart required.

---

# 🛠 Production Engineering Features

## 🔋 Power Suspend Recovery

Factory PCs sleep.
Operators unplug cameras.
Windows resumes unpredictably.

SYSTEM survives all of it.

### Recovery System

* Win32 `WM_POWERBROADCAST`
* Background listener thread
* Graceful camera shutdown
* Automatic reconnect on resume

```text
PC sleeps → Camera closes safely
PC resumes → Camera reconnects automatically
```

Zero operator intervention.

---

## 🔥 Live Excel Hot Reload

Operators can edit Excel databases while the app is running.

```text
Excel modified
      ↓
QFileSystemWatcher detects change
      ↓
Database reloads automatically
```

### Supports

* Part code updates
* New operator names
* Runtime refresh
* 100ms debounce protection

---

## ↩ Undo Stack

Every count operation stores full application state.

```python
history.push(counter_state)
```

Undo restores:

* Part quantity
* Overall quantity
* Pallet totals
* LCD counters
* Judgement state

Atomically.

---

## ⌨ Operator-Friendly UI

Because factory software must work for real humans.

### Features

✅ Touchscreen support
✅ On-screen keyboard
✅ Smart autocomplete
✅ Color-coded counters
✅ Fullscreen operation
✅ Large buttons for gloves/operators

---

# 🧱 System Architecture

## Project Structure

```text
pallet_system/
│
├── main.py
│   └── Application bootstrap + model loading
│
├── code/
│   ├── GUI.py
│   ├── camera_wrapper.py
│   ├── camera_worker.py
│   ├── yolo_infer.py
│   ├── actions.py
│   └── exceptions.py
│
├── utils/
│   ├── file_utils.py
│   └── keyboard_utils.py
│
├── SetRecord/
│   └── Runtime Excel databases
│
└── runs/
    └── YOLO training outputs
```

---

## Runtime Pipeline

```text
┌──────────────────────────────┐
│     Hikrobot Industrial      │
│           Camera             │
│      (GigE / USB3 Vision)    │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│      camera_wrapper.py       │
│                              │
│  - MVS SDK interface         │
│  - Camera initialization     │
│  - Stream handling           │
│  - Auto reconnect            │
│  - Power event recovery      │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│      camera_worker.py        │
│                              │
│  - Frame grabbing            │
│  - Preprocessing             │
│  - Brightness adjustment     │
│  - Detection pipeline call   │
│  - QThread frame updates     │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│        yolo_infer.py         │
│                              │
│  Detection Pipeline          │
│                              │
│  1. YOLOv8 Detection         │
│  2. Grid Gap Inference       │
│  3. OpenCV Validation        │
│                              │
│  - Bounding box generation   │
│  - Missing slot estimation   │
│  - IOU filtering             │
│  - Overlay rendering         │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│            GUI.py            │
│                              │
│  - Live camera display       │
│  - Bounding box overlays     │
│  - Counter display           │
│  - Pallet mode switching     │
│  - Operator controls         │
│  - Result judgement          │
└───────┬───────────┬──────────┘
        │           │
        │           ▼
        │   ┌──────────────────┐
        │   │    actions.py    │
        │   │                  │
        │   │  - Count         │
        │   │  - Save          │
        │   │  - Undo          │
        │   │  - Reset         │
        │   └────────┬─────────┘
        │            │
        ▼            ▼
┌──────────────────────────────┐
│      Data + File System      │
│                              │
│  - Excel logging             │
│  - Screenshot capture        │
│  - QFileSystemWatcher        │
│  - Runtime database reload   │
└──────────────────────────────┘
```
---

# ⚡ Getting Started

## Requirements

* 🪟 Windows 10/11
* 📷 Hikrobot GigE / USB3 camera
* 🐍 Python 3.10+
* 🧠 Hikrobot MVS SDK

---

### Workflow

1️⃣ Launch app
2️⃣ Select pallet mode
3️⃣ Camera starts automatically
4️⃣ Press **COUNT**
5️⃣ Press **SAVE**
6️⃣ Excel + screenshot generated

---

# 🧪 Model Training

Weights are excluded from the repository.

Train your own:

```bash
yolo train \
    model=yolov8n.pt \
    data=your_data.yaml \
    epochs=200
```

Place trained weights here:

```text
runs/train100_200_epochs/weights/best.pt
```

---

# 📂 Output Structure

```text
📁 SetRecord/
└── 2025/
    └── May 25/
        └── RESULT.xlsx

📁 PhotoRecord/
└── 2025/
    └── May/
        └── 2025-05-14/
            └── PARTCODE_LOT_TIMESTAMP.jpg
```

---

# 📊 Excel Output

Each save operation logs:

* 📅 Date / Time
* 🏭 Pallet Type
* 🔢 Lot Number
* 🧩 Part Code
* 🆔 Pallet ID
* 👤 Operator Name
* 📦 Part Quantity
* 📈 Overall Quantity
* 🧮 Total Pallets
* ✅ Final Judgement

---

# ⚙ Tech Stack

<div align="center">

| Category               | Technology                    |
| ---------------------- | ----------------------------- |
| 🐍 Language            | Python 3.10+                  |
| 🧠 AI Detection        | YOLOv8                        |
| 👁 Computer Vision     | OpenCV                        |
| 🖥 GUI                 | PyQt5                         |
| 📷 Camera SDK          | Hikrobot MVS                  |
| 📊 Excel I/O           | pandas + openpyxl             |
| 🔍 Secondary Detection | HoughCircles + Blob Detection |
| 📦 Deployment          | PyInstaller                   |

</div>

---

# 💡 Key Engineering Lessons

### 🧠 AI alone is not enough

Structured industrial environments benefit massively from hybrid systems:

```text
Neural Detection + Spatial Logic + Rule Validation
```

---

### ⚙ Hardware integration is hard

Industrial cameras introduce:

* driver failures
* suspend/resume issues
* packet loss
* timing edge cases

Robust recovery logic matters as much as ML accuracy.

---

### 👷 Real operators shape the product

Features like:

* autocomplete
* undo
* giant buttons
* touchscreen support
* visual feedback

weren't “nice-to-have”.

They were essential.

---

# 🌟 Future Improvements

* 📡 Multi-camera synchronization
* ☁ Centralized production dashboard
* 📈 Statistical defect analytics
* 🧠 Active-learning retraining pipeline
* 📱 Remote monitoring interface
* 🔔 Real-time alert notifications

---

<div align="center">

# 🏁 SYSTEM

### *Industrial AI built for the real world.*

Built with ❤️ by **Your Name**

[LinkedIn](https://linkedin.com/in/yourprofile) • [Portfolio](https://yoursite.com)

---

</div>
