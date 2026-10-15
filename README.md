# DentalOPG-V1-Initial-Prototype

An early research prototype for screening **2D panoramic dental X-rays (OPGs)** using pretrained U-Net tooth-region segmentation, quadrant-based features, Random Forest classification, and a Flask visualization dashboard.

> **Research prototype only.** Outputs are exploratory screening indicators, not clinical diagnoses or validated treatment recommendations.

## Overview

The project processes an OPG image, estimates a tooth-region mask, calculates the fraction of segmented pixels in four image quadrants, and uses these features for classification. A browser-based dashboard shows missing-teeth and wisdom-teeth predictions alongside quadrant percentages, a left/right symmetry heuristic, a radiopacity-based anatomy heuristic, and an annotated image.

## Pipeline

```text
OPG image (grayscale)
    -> resize to 512 x 512, normalize to [0, 1]
    -> pretrained U-Net prediction, threshold > 0.5
    -> resize binary mask to original image dimensions
    -> quadrant percentages (Q1/Q2/Q3/Q4)
    -> Random Forest prediction
    -> Flask dashboard + segmented-contour overlay + heuristic insights
```

Quadrants follow the displayed anatomical orientation: Q1 upper right, Q2 upper left, Q3 lower left, and Q4 lower right. Each feature is the percentage of all segmented tooth-mask pixels located in that quadrant, **not a clinically calibrated bone/tooth density measurement**.

## Features

- Pretrained U-Net tooth-region segmentation.
- Four quadrant mask-percentage features.
- Random Forest classifiers with 500 trees and `class_weight="balanced"`.
- Dashboard predictions for `Missing_Teeth` and `Wisdom_Teeth_Status`.
- Left/right mask-distribution symmetry heuristic.
- Threshold-based radiopacity heuristics for possible restorative/anatomical findings; **not validated restoration detection**.
- Original-versus-annotated OPG comparison in a Flask web interface.

## Data and evaluation

- The local label CSV contains **116 image records** and the columns `Image`, `Missing_Teeth`, `Wisdom_Teeth_Status`, `Impaction_Type`, and `Alignment_issue`.
- `train_v3.py` extracts quadrant features and trains a **separate Random Forest for each of the four targets**, using an 80/20 train/test split (`random_state=42`, stratified when applicable).
- It prints accuracy and a classification report for each target and saves `saved_models/opg_bundle_v3.pkl`.
- The dashboard uses **only the missing-teeth and wisdom-teeth classifiers**.
- This is the **V1 holdout-split implementation**, not the later five-fold cross-validation project. No performance numbers are asserted here without the corresponding saved evaluation output.

## Project scripts

| File | Role |
| --- | --- |
| `train_v3.py` | U-Net-based feature extraction, four-target RF training and holdout evaluation |
| `web_dashboard.py` | Flask UI, two RF predictions, heuristic insights and OPG visualization |
| `evaluate_rf_model_accuracy.py` | Evaluation utility (interpret its results separately from holdout evaluation) |
| `inspect_rf_model.py` | RF model inspection utility |
| `model/extract_model.py` | Model-related utility |
| `model/inspect_model.py` | Model-related utility |

## Environment

The original working Mac environment used Python **3.8.10** and the following observed packages:

- `tensorflow-macos==2.13.0`
- `keras==2.13.1`
- `Flask==3.0.3`
- `opencv-python==4.13.0.92`
- `numpy==1.24.3`
- `pandas==2.0.3`
- `scikit-learn==1.3.2`
- `joblib==1.4.2`

See `requirements.txt` for the recorded versions. **Fresh-environment installation compatibility has not been independently verified**; in particular, verify Python/platform compatibility of OpenCV and TensorFlow before attempting a new install. `tensorflow-macos` is macOS-specific.

## Local setup and execution

1. Use a compatible Python environment and install tested dependencies (adjust versions if necessary for your platform).
2. Supply the pretrained U-Net model at `model/dental_xray_seg.h5`.
3. For inference, supply the compatible trained RF bundle at `saved_models/opg_bundle_v3.pkl`.
4. Run from the repository root:

   ```bash
   python web_dashboard.py
   ```

5. Open `http://127.0.0.1:5002` and upload an OPG image for analysis.

To **retrain** instead, place the authorized OPG images under `images/` and provide `dataset_v3_final_116.csv` with the expected image IDs and label columns, then run:

```bash
python train_v3.py
```

The training script also writes `dataset_v3_with_features.csv`. All relative paths assume the repository root is the working directory. Use the interpreter executable path instead of `python` if your shell does not resolve the active environment.

## Files not included in the public repository

The repository intentionally excludes local environments, medical images, uploads, generated outputs, local dataset CSVs, and the large pretrained U-Net weights. Serialized RF bundles may also be excluded pending provenance and distribution review. **A fresh clone cannot perform inference without the compatible pretrained U-Net and RF bundle.** Obtain model weights and data only from sources with appropriate access and redistribution permissions; no download source is claimed here.

## Limitations

- Small original dataset (116 labeled image records).
- Image-level, quadrant-based features rather than individual tooth-instance or FDI localization.
- Holdout evaluation; no five-fold cross-validation is implemented in `train_v3.py`.
- Symmetry and radiopacity findings are heuristic, not independently validated clinical diagnoses.
- Model/data availability and original pretrained model provenance must be documented before claiming fully reproducible training or inference.
- The development Flask server uses `debug=True`; run locally only and disable debug for any deployment.

## Project evolution

This repository preserves an **early OPG analysis prototype**. Subsequent projects explored stronger evaluation, tooth-instance segmentation, FDI assignment, and restoration analysis. Results from those later projects should not be attributed to this V1 codebase.
