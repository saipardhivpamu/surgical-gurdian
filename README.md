# surgical-gurdian
AI-powered laparoscopic surgery safety monitoring system using YOLOv8 &amp; OpenCV. Detects surgical tools in real-time, tracks proximity with 3-tier alerts (Caution/Warning/Critical), motion trails, velocity analysis, CLAHE enhancement, and auto-generates session safety reports.

# 🏥 Surgical Guardian v4

> **Real-time AI-powered laparoscopic surgery safety monitoring system**  
> Detects surgical tools and critical vessels using YOLOv8, triggers tiered proximity alerts, tracks instrument motion, and auto-generates post-session safety reports.

[![Python](https://img.shields.io/badge/Python-3.9+-blue?logo=python&logoColor=white)](https://python.org)
[![YOLOv8](https://img.shields.io/badge/YOLOv8-Ultralytics-orange)](https://ultralytics.com)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.x-green?logo=opencv&logoColor=white)](https://opencv.org)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20Linux%20%7C%20macOS-lightgrey)]()

---

## 📌 Overview

**Surgical Guardian v4** is built with one primary mission — preventing accidental damage to blood vessels during laparoscopic surgery. It runs a YOLOv8 model (`best.pt`) on live camera feeds or recorded video to:

- Detect surgical tools, organs, and critical vessels (Cystic Artery, Cystic Duct) in real time
- Compute proximity distances between tools and vessels
- Trigger **3-tier alerts** (Caution → Warning → Critical) with audio beeps and visual overlays
- Track instrument motion trails and approach velocity
- Log all alert events to CSV and generate a post-session safety score report

---

## 🗂️ Project Structure

```
surgical-guardian/
├── surgical_guardian_v4.py   # Main application — run this
├── best.pt                   # Trained YOLOv8 model weights
├── sg_output/                # Auto-created at runtime
│   ├── session_YYYYMMDD_HHMMSS.mp4         # Annotated session video
│   ├── alert_log_YYYYMMDD_HHMMSS.csv       # Per-frame alert events
│   └── session_report_YYYYMMDD_HHMMSS.txt  # Safety score report
└── README.md
```

---

## 🔍 What the Model Detects

The `best.pt` model is trained on laparoscopic cholecystectomy footage and detects **16 classes**:

| Category | Classes |
|---|---|
| 🔧 **Surgical Tools** | `bipolar`, `clipper`, `grasper`, `hook`, `irrigator`, `scissors`, `specimen_bag` |
| 🫀 **Organs** | `liver`, `gallbladder`, `abdominal_wall`, `fat`, `GI_tract`, `connective_tissue`, `liver_ligament` |
| ⚠️ **Critical Vessels** | `CYSTIC ARTERY`, `CYSTIC DUCT` |

> Proximity alerts are triggered **only** between surgical tools and critical vessels.

---

## ⚡ Key Features

### 🔴 3-Tier Proximity Alert System

| Tier | Distance Threshold | Audio | Visual |
|---|---|---|---|
| **CAUTION** | < 150 px | 600 Hz beep | Yellow overlay |
| **WARNING** | < 100 px | 800 Hz beep | Orange flash |
| **CRITICAL** | < 60 px | 1100 Hz beep | Red border + 7-frame freeze |

### 🧠 Per-Tool Danger Weighting
Each tool has an assigned danger weight that adjusts its effective proximity distance — riskier tools trigger alerts earlier:

```
hook         → 1.0  (highest risk — dissects near vessels)
clipper      → 0.9  (clips vessels, high risk if misplaced)
scissors     → 0.85 (cuts, high risk)
bipolar      → 0.8  (cuts & coagulates)
grasper      → 0.4  (blunt, lower risk)
irrigator    → 0.2  (fluid, low risk)
specimen_bag → 0.1  (retrieval, minimal risk)
```

### 🖼️ CLAHE Image Enhancement
Applies Contrast Limited Adaptive Histogram Equalization on the L-channel (LAB color space) to reduce laparoscopic smoke and fog haze before inference.

### 🌀 Motion Trail Tracking
Maintains a 25-frame deque of tool center positions and renders fading color trails to visualize instrument movement paths.

### 📐 IOU-Based Temporal Smoothing
Exponential smoothing of bounding box coordinates across frames using Intersection-over-Union matching — eliminates detection flicker.

### 📊 Velocity & Approach Analysis
Computes per-tool velocity vectors and approach rate toward vessels. Flags tools actively moving toward critical structures even before a distance threshold is breached.

### 🏥 Organ Overlap Warning
Detects when a tool center point falls inside an organ bounding box and displays a live warning banner on-screen.

### 📋 CSV Alert Logging
Every proximity alert event is logged with:
```
timestamp, frame, level, tool, confidence, vessel, distance_px, approach_rate, danger_weight
```

### 📄 Auto Safety Score Report
On session end, auto-generates a `.txt` report with:
- Alert counts by tier (Caution / Warning / Critical)
- Closest approach distance ever recorded
- Safety score (0–100) and assessment rating

---

## 🚀 Installation

### Prerequisites
- Python 3.9+
- A webcam, DroidCam, or a `.mp4` video file

### 1. Clone the Repository
```bash
git clone https://github.com/<your-username>/surgical-guardian.git
cd surgical-guardian
```

### 2. Install Dependencies
```bash
pip install ultralytics opencv-python numpy
```

### 3. Add the Model File
Place `best.pt` in the root of the project folder (same directory as `surgical_guardian_v4.py`).

---

## ▶️ Usage

### Default — DroidCam over Wi-Fi
```bash
python surgical_guardian_v4.py
```

### Webcam
```bash
python surgical_guardian_v4.py --source 0
```

### Video File (Demo / Expo Mode)
```bash
python surgical_guardian_v4.py --source path/to/video.mp4
```

### Custom Model & Confidence Threshold
```bash
python surgical_guardian_v4.py --model best.pt --conf 0.35
```

### Disable Recording or Logging
```bash
python surgical_guardian_v4.py --no-record --no-log
```

### All CLI Arguments

| Argument | Default | Description |
|---|---|---|
| `--source` | DroidCam URL | Video source: URL, file path, or webcam index |
| `--model` | `best.pt` | Path to YOLOv8 `.pt` model file |
| `--conf` | `0.30` | Initial detection confidence threshold (0–1) |
| `--width` | `640` | Frame width in pixels |
| `--height` | `480` | Frame height in pixels |
| `--no-record` | — | Disable session video recording |
| `--no-log` | — | Disable CSV alert logging |

---

## ⌨️ Keyboard Controls

| Key | Action |
|---|---|
| `Q` / `ESC` | Quit and save session report |
| `P` | Pause / Resume |
| `S` | Save screenshot |
| `R` | Reset session stats |
| `+` / `-` | Increase / Decrease confidence threshold live |

---

## 📊 Output Files

All outputs are saved to `sg_output/` (auto-created on first run):

```
sg_output/
├── session_20250522_143012.mp4         ← Annotated session video
├── alert_log_20250522_143012.csv       ← Per-event alert log
└── session_report_20250522_143012.txt  ← Safety score report
```

### Sample Safety Report Output
```
══════════════════════════════════════════════════════════════
  SURGICAL GUARDIAN v4 — SESSION SAFETY REPORT
══════════════════════════════════════════════════════════════
  Date/Time   : 2025-05-22  14:30:12
  Source      : demo_surgery.mp4
  Model       : best.pt
  Duration    : 04:32
  Frames proc : 5440
──────────────────────────────────────────────────────────────
  PROXIMITY EVENTS
  Total alerts    : 8
  CRITICAL (<60px): 1
  WARNING  (<100px): 3
  CAUTION  (<150px): 4
  Closest approach: 42 px
──────────────────────────────────────────────────────────────
  SAFETY ASSESSMENT
  Safety score    : 79/100
  Assessment      : GOOD
══════════════════════════════════════════════════════════════
```

---

## 🏗️ System Architecture

```
Video Source (Webcam / File / DroidCam / RTSP)
        │
        ▼
  CLAHE Enhancement ───────── reduces laparoscopic haze
        │
        ▼
  YOLOv8 Inference (best.pt)
        │
        ▼
  IOU Temporal Smoothing ───── eliminates bbox flicker
        │
        ├──► Tool Detection  →  Motion Trails + Velocity Arrows
        ├──► Organ Detection →  Overlap Warning
        └──► Vessel Detection → Proximity Analysis
                                      │
                    ┌─────────────────┼──────────────────┐
                    ▼                 ▼                   ▼
               CAUTION             WARNING            CRITICAL
             (< 150 px)          (< 100 px)          (< 60 px)
             600 Hz beep         800 Hz beep        1100 Hz beep
                                                   + Frame Freeze
                                      │
        ┌─────────────────────────────┘
        ▼
  HUD Overlay + CSV Logger + Session Video Recorder
        │
        ▼
  Auto Safety Score Report (saved on exit)
```

---

## 🎓 Academic Context

Developed as a B.Tech CSE (AI & ML) capstone project at  
**ACE Engineering College (Autonomous), Hyderabad**

**Core Technologies:**
- [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics) — real-time object detection
- [OpenCV](https://opencv.org/) — image processing, visualization, video I/O
- [NumPy](https://numpy.org/) — numerical computation
- Python `threading` — latency-free frame capture and async audio alerts

---


---

# Acknowledgements

- [Ultralytics](https://ultralytics.com/) for the YOLOv8 framework
- [CholecT50 Dataset](https://github.com/CAMMA-public/cholect50) — inspiration for surgical instrument detection classes
- ACE Engineering College, Hyderabad — academic support and resources

---

> ⚠️ **Disclaimer:** Surgical Guardian is a research prototype for academic demonstration purposes only. It is **not** a certified medical device and must **not** be used in real clinical or surgical environments without proper regulatory approval.
