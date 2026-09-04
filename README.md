# ⚡ JutsuCam — Real-Time Anime Hand-Seal VFX Engine

[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![MediaPipe](https://img.shields.io/badge/MediaPipe-0.10.14-orange.svg)](https://developers.google.com/mediapipe)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.9+-green.svg)](https://opencv.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

**JutsuCam** is a real-time computer vision application that recognizes iconic anime hand seals (mudras / jutsu signs) via webcam or pre-recorded video feeds and dynamically overlays procedural, physics-driven visual effects (VFX) directly onto the user's hands.

Built entirely with standard OpenCV primitives and deterministic procedural math (no external artwork, pre-rendered green screen clips, or heavy neural nets required for rendering), JutsuCam achieves seamless **30+ FPS** performance on standard CPU hardware.

---

## 🌟 Key Features

- **Real-Time Hand Tracking**: Leverages Google MediaPipe Hands (21 3D landmarks per hand, multi-hand detection) operating in camera pixel coordinates.
- **Rule-Based Geometric Gesture Classifier**:
  - Deterministic angle-based finger curl calculation ($0^{\circ}$ to $180^{\circ}$).
  - Fingertip spread ratios, thumb splay analysis, wrist orientation, and multi-hand Euclidean proximity checks.
  - Zero black-box neural network training required for gesture definitions—fully auditable and calibrated via `src/config.py`.
- **Temporal Debouncing & Cooldown State Machine**:
  - Requires continuous gesture retention for a configurable hold threshold (`GESTURE_HOLD_FRAMES = 6`) to eliminate transient noise/flicker.
  - Built-in cooldown timer (`GESTURE_COOLDOWN_FRAMES = 45`) preventing infinite re-triggering while holding a pose.
- **100% Procedural VFX Engine**:
  - **Rasengan**: Spinning multi-layer concentric rotating arcs with additive radial luminance glow.
  - **Chidori**: Procedurally generated jagged lightning branches randomized per-frame.
  - **Shadow Clone (Kage Bunshin)**: Frame-buffer region capture producing dual semi-transparent ghost clones flanking the user.
  - **Fire Dragon**: Bézier-curve trajectory flame particle streaming with dynamic color gradient decay (yellow $\to$ orange $\to$ red $\to$ ash).
  - **Water Dragon**: Fluid particle trail following curved cubic sweeps with oceanic color gradients.
  - **Earth Release**: Ascending layered polygonal rock strata pillars growing from the ground/hand anchor.
- **Production-Ready Data & Reporting Pipeline**:
  - Flat CSV event persistence (`data/logs/jutsu_session_log.csv`) without external database overhead.
  - Automated session analytics and headless chart generation (`matplotlib`) reporting execution counts, most frequent seals, and trigger timelines.
- **Headless CI & Video Batch Mode**:
  - Native video file input support (`--source video.mp4`).
  - Headless rendering (`--headless --output out.mp4`) designed for CI/CD automated validation and server environments.
  - Deterministic synthetic pose fixtures for lightning-fast, camera-less unit tests.

---

## 🥋 Supported Hand Signs & Jutsu

| Jutsu | Japanese Sign | Hand Pose & Landmark Mechanics | Visual Effect (OpenCV Procedural) |
| :--- | :--- | :--- | :--- |
| **Rasengan** | *Spiraling Sphere* | **1 Hand**: Cupped palm facing forward, fingers partially curled ($90^{\circ}-150^{\circ}$) with tips widely spread. | Concentric swirling blue-white particle orbs with high-intensity additive center. |
| **Chidori** | *One Thousand Birds* | **1 Hand**: Open flat palm facing camera, all 5 fingers extended and spread outward. | White-cyan crackling electrical bolts radiating dynamically from palm. |
| **Shadow Clone** | *Kage Bunshin no Jutsu* | **2 Hands**: Crossed wrists forming a plus seal; index fingers extended straight up, remaining fingers curled into fists. | Semi-transparent offset alpha-blended clone snapshots on left & right. |
| **Fire Dragon** | *Karyū Endan* | **1 Hand**: Tiger seal (index + middle fingers extended together, ring/pinky curled) with **thumb extended outward** at a diagonal tilt. | Sinuous Bézier flame stream surging outward with ember dispersion. |
| **Water Dragon** | *Suiryūdan no Jutsu* | **1 Hand**: Ox seal (index + middle fingers extended together, ring/pinky curled) with **thumb tucked flat**, hand upright. | Sweeping fluid arc of azure and cyan droplets trailing the wrist. |
| **Earth Release** | *Doryūheki* | **1 Hand**: Tight closed fist pressed downward, low fingertip spread, fingertips pointing down. | Jagged brown-slate rock pillar strata rising dynamically from lower anchor. |

---

## 🏛️ System Architecture

```text
               +-----------------------------------+
               |        Video Input Source         |
               | (Webcam 0/1 or Saved MP4 Video)   |
               +-----------------+-----------------+
                                 | BGR Frame (960x540)
                                 v
               +-----------------------------------+
               |      Module 1: HandTracker        |
               |    (MediaPipe Hands Solutions)    |
               +-----------------+-----------------+
                                 | 21 (x, y, z) Landmarks per hand
                                 v
               +-----------------------------------+
               |   Module 2: GestureClassifier     |
               |  Geometric Angles + Rule Engine   |
               |     + Temporal Hold State Machine |
               +-----------------+-----------------+
                                 | Trigger Event / Hand Anchor (x, y)
                  +--------------+--------------+
                  |                             |
                  v                             v
+---------------------------------+  +--------------------------------+
|      Module 4: Persistence      |  |     Module 3: VFX Engine       |
|  - SessionLogger (CSV Events)   |  |  - Active Effects Registry     |
|  - ReportGenerator (Matplotlib) |  |  - Procedural Math & Blending  |
+---------------------------------+  +----------------+---------------+
                                                      | Composited Frame
                                                      v
                                     +--------------------------------+
                                     |   HUD Overlay & Render Output  |
                                     |  (cv2.imshow / headless MP4)   |
                                     +--------------------------------+
```

---

## 📁 Repository Structure

```text
jutsucam/
├── assets/                     # Media, reference illustrations, badges
├── data/
│   ├── logs/                   # Event logs (jutsu_session_log.csv) & sample test video
│   └── reports/                # Generated analytics reports (summary.csv, charts)
├── diagrams/                   # Architecture and pipeline diagrams
├── src/
│   ├── effects/                # Procedural visual effect implementations
│   │   ├── __init__.py         # Effect class registry mapping
│   │   ├── base_effect.py      # Abstract BaseEffect class contract
│   │   ├── chidori.py          # Jagged lightning simulation
│   │   ├── earth_release.py    # Rising stone pillar strata
│   │   ├── fire_dragon.py      # Bézier flame particle engine
│   │   ├── rasengan.py         # Swirling additive energy sphere
│   │   ├── shadow_clone.py     # Alpha-composited frame buffer duplicates
│   │   └── water_dragon.py     # Hydrodynamic droplet curves
│   ├── __init__.py
│   ├── config.py               # Tunable constants, thresholds, and jutsu registry
│   ├── gesture_classifier.py   # Geometric feature extraction & state machine
│   ├── hand_tracker.py         # MediaPipe Hands wrapper
│   ├── logger_setup.py         # Standardized project logger configuration
│   ├── report_generator.py     # Matplotlib session analytics & CSV exporter
│   ├── session_logger.py       # Flat CSV event recorder
│   ├── utils.py                # FPS counter, HUD renderer, vector helpers
│   └── vfx_engine.py           # Particle lifecycle manager and compositor
├── tests/
│   ├── manual_checks/
│   │   └── synth_poses.py      # Synthetic HandLandmarks generators for testing
│   ├── __init__.py
│   ├── fixtures.py             # Shared pytest fixtures (no camera required)
│   ├── test_gesture_classifier.py
│   ├── test_session_logger.py
│   └── test_vfx_engine.py
├── main.py                     # CLI entry point (run, report)
├── requirements.txt            # Pinned dependencies
├── LICENSE                     # MIT License
├── .gitignore
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- **Python 3.9 – 3.11** (recommended)
- Webcam (built-in or USB external) or sample video file
- OS: Linux, macOS, or Windows

### 1. Installation

```bash
# Clone the repository
git clone https://github.com/your-username/jutsucam.git
cd jutsucam

# Create and activate virtual environment
python -m venv venv

# On Linux / macOS:
source venv/bin/activate
# On Windows:
# venv\Scripts\activate

# Install pinned dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

> **Note on MediaPipe version**: `requirements.txt` pins `mediapipe==0.10.14`. This ensures the built-in `mp.solutions.hands` API and bundled offline model weights are used directly, avoiding runtime model downloads from external cloud storage.

---

## 🎮 CLI Usage

The CLI entry point is `main.py`, providing subcommands for live tracking, video processing, and reporting:

### 1. Live Webcam Session

Launch live tracking using default camera index `0`:

```bash
python main.py run --source 0
```

- To switch cameras, pass `--source 1` (or your secondary device index).
- Press **`q`** at any time inside the video window to safely exit and save session logs.

### 2. Processing Pre-Recorded Videos

Evaluate the pipeline deterministically on any video file:

```bash
python main.py run --source data/logs/synthetic_test.mp4
```

### 3. Headless Processing (No GUI / CI Pipeline)

For headless servers, remote cloud instances, or CI workflows without an X-server / graphical display:

```bash
python main.py run --source data/logs/synthetic_test.mp4 --headless --output data/logs/session_output.mp4
```

- `--max-frames <N>`: Optionally terminate execution after $N$ frames (e.g. `--max-frames 120` for quick automated sanity runs).

### 4. Generate Analytics & Usage Reports

Inspect trigger statistics across all historical sessions:

```bash
python main.py report
```

Output includes:
- Total sessions & trigger counts printed to terminal.
- Exported detailed breakdown in `data/reports/summary.csv`.
- Bar chart graphic generated at `data/reports/jutsu_usage.png`.

---

## 🧪 Running Tests

The test suite uses **synthetic landmark geometry** modeled after real hand poses, meaning unit tests execute in milliseconds without requiring camera hardware or network access:

```bash
# Run entire test suite
pytest -v

# Run gesture classification unit tests
pytest tests/test_gesture_classifier.py -v

# Run VFX lifecycle and rendering integrity tests
pytest tests/test_vfx_engine.py -v

# Run session logging tests
pytest tests/test_session_logger.py -v
```

---

## ⚙️ Configuration & Calibration

All geometric thresholds, visual dimensions, and timing settings are centralized in `src/config.py`:

```python
# Video capture
FRAME_WIDTH = 960
FRAME_HEIGHT = 540
FLIP_HORIZONTAL = True          # Mirrors webcam for natural UX

# Gesture sensitivity
GESTURE_HOLD_FRAMES = 6         # Frames a seal must be sustained before triggering
GESTURE_COOLDOWN_FRAMES = 45    # Frame delay before the same seal can fire again
FINGER_EXTENDED_ANGLE_DEG = 150 # Angle threshold between straight (>150°) vs bent (<150°)

# VFX parameters
RASENGAN_MAX_RADIUS = 70
CHIDORI_BOLT_COUNT = 14
SHADOW_CLONE_ALPHA = 0.45
FIRE_DRAGON_SEGMENTS = 18
```

---

## 🤝 Contributing

Contributions are welcome! Feel free to:
1. Fork the repository.
2. Create a feature branch (`git checkout -b feature/new-jutsu-effect`).
3. Add your procedural effect class in `src/effects/` inheriting from `BaseEffect`.
4. Register the new jutsu in `src/config.py` and rule in `src/gesture_classifier.py`.
5. Add unit test fixtures in `tests/test_gesture_classifier.py`.
6. Commit changes and submit a Pull Request.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
