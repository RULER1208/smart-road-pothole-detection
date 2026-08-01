# Smart Road Pothole Detection System

Detecting and measuring road potholes with three deep-learning object detectors,
compared on the same dataset and delivered as an interactive Streamlit app.

**Course:** BMCS2003 Artificial Intelligence — Title 6: Image Processing and Computer Vision

---

## Overview

Potholes are usually found through manual inspection, which is slow and
inconsistent. This project trains three detectors on the same annotated pothole
dataset, compares them on a held-out test set, and deploys the strongest model
in a web application that detects potholes in images and video, estimates their
size, and counts unique potholes in a video without recounting them.

## Results

Evaluated once on the held-out test split: **198 images, 433 labelled potholes**.
Model selection used the validation split. P/R/F1 measured at confidence 0.50
and IoU 0.50.

| Model | mAP@0.5 | mAP@0.5:0.95 | Precision | Recall | F1 | FPS | Params |
|---|---|---|---|---|---|---|---|
| **YOLOv8n** | **0.914** | **0.621** | **0.891** | 0.815 | **0.852** | **120** | **3.0 M** |
| Faster R-CNN ResNet50-FPN | 0.885 | 0.510 | 0.697 | **0.898** | 0.785 | 15.4 | 41.3 M |
| SSD300-VGG16 | 0.808 | 0.488 | 0.880 | 0.674 | 0.763 | 49.0 | 23.7 M |

**YOLOv8n is the best model**: highest accuracy, highest F1, fastest inference,
and the fewest parameters. Faster R-CNN achieves the highest recall but the
lowest precision (169 false positives) and cannot sustain real-time video.

Speed was measured with batch size 1 on a single T4 GPU, timing from a decoded
in-memory image through preprocessing, transfer, inference and NMS, and
excluding disk reading.

> **Note on confidence scores.** YOLOv8 applies a sigmoid to class logits while
> SSD and Faster R-CNN apply a softmax over background-versus-pothole, so raw
> confidence values are not comparable across architectures. Faster R-CNN
> reports the highest confidences yet has the lowest precision. Models are
> ranked by verified metrics, not by confidence.

## Features

- Image detection with all three models side by side
- Video detection with unique pothole counting (IoU tracking + counting line)
- Pothole size estimation in pixels, percentage of frame, or centimetres
- Three severity tiers (Small / Medium / Large) with a warning banner
- CSV export of every detection, with timestamps and track IDs for video
- Benchmark tab showing the full test-set comparison

## Quick start

```bash
git clone https://github.com/<your-username>/smart-road-pothole-detection.git
cd smart-road-pothole-detection

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt

# Download the three checkpoints from Releases into models/
streamlit run ui.py
```

The app opens at `http://localhost:8501`. A GPU is optional — the app runs on
CPU, more slowly.

## Repository structure

```
.
├── ui.py                     # Streamlit application
├── requirements.txt
├── .streamlit/config.toml    # Theme
├── models/                   # Checkpoints (downloaded, not committed)
├── notebooks/                # Training, evaluation, preprocessing
├── docs/                     # Design system, screenshots
└── sample_data/              # A few demo images
```

## Dataset

Roboflow Universe "pothole" dataset (CC BY 4.0): 3,940 images at 640×640 with
10,111 annotated potholes, split 3,345 train / 397 validation / 198 test.

Preprocessing applied: auto-orientation, resize to 640×640, and flip
augmentation on the training split. Verification confirmed 0 missing labels,
0 corrupt images, 0 invalid boxes, and 0 duplicate images leaking from the
training split into validation or test.

## Method

1. **Preprocessing** — verify the dataset, compute statistics, check for leakage
2. **Training** — each model trained to convergence on the identical split;
   the best checkpoint selected on the validation set
3. **Evaluation** — a single pass over the test split with identical metrics
   and an identical speed-measurement method for all three models
4. **Deployment** — the best model served in the Streamlit app

## Limitations

- Bounding boxes are rectangles, so size figures estimate the box rather than
  the true irregular pothole outline; accurate area requires segmentation
- Centimetre estimates need a calibration marker and hold only at the
  calibrated camera distance and angle
- Unique video counting is an estimate; camera shake, occlusion or a static
  camera can affect it
- Results come from one dataset and one training run per model

## Acknowledgements

Dataset from Roboflow Universe. Models built with Ultralytics YOLOv8 and
Torchvision. UI design system generated with `ui-ux-pro-max`.
