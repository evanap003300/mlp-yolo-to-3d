# Drone Tracking: 3D Coordinate Conversion from YOLO Predictions

This repository contains the code, data, models, and published paper for a workflow that converts 2D YOLO drone detections from a fisheye camera into 3D OptiTrack-aligned coordinates using a Multi-Layer Perceptron (MLP).

## Paper

The final paper is included in this repository:

**[paper/MLPUAV-SPIE_V1.pdf](paper/MLPUAV-SPIE_V1.pdf)**

## Project Structure

```
Converting_YOLO_Coordinates_To_3D/
├── data/
│   ├── raw/            # Raw YOLO and OptiTrack spreadsheets
│   ├── processed/      # Pixel-to-mm converted YOLO coordinates and curated model input
│   ├── predictions/    # MLP predictions output
│   └── plots/          # Generated figures
├── models/             # Trained MLP model and input/output scalers
├── src/
│   ├── preprocessing/  # align_and_format.py — pixel-to-mm conversion
│   ├── modeling/       # train_model.py, predict_and_export.py
│   └── plotting/       # plot_results.py
├── making_figures/     # Self-contained folders for reproducing each paper figure
│   ├── Figure1_Drone/
│   ├── Figure2_EPM/
│   ├── Figure3_3D_Trajectory/
│   ├── Figure4_Error_Distribution/
│   ├── Figure5_Scatter_Plots/
│   └── Figure6_Error_Over_Time/
├── paper/              # Final published paper (PDF)
├── requirements.txt
└── LICENSE.txt
```

## Workflow

### 1. Preprocessing — `src/preprocessing/align_and_format.py`
Loads `data/raw/YOLO_Coordinates.xlsx`, converts normalized pixel coordinates to millimeters, and writes `data/processed/YOLO_Coordinates_mm.xlsx`.

> The combined `input_coordinates.xlsx` used for training and prediction is a manually curated subsection of `YOLO_Coordinates_mm.xlsx` and `OptiTrack_Cameras.xlsx` (gaps exist where the drone leaves the frame). The curated file is provided directly in `data/processed/` and in each `making_figures/` subfolder so results are reproducible without repeating the manual step.

### 2. Model training — `src/modeling/train_model.py`
Trains an MLP with RobustScaler normalization, batch normalization, dropout, L2 regularization, and Huber loss. Saves `mlp_converter.keras`, `x_scaler.pkl`, and `y_scaler.pkl` to `models/`.

### 3. Prediction — `src/modeling/predict_and_export.py`
Loads the trained model and scalers, predicts 3D coordinates from `input_coordinates.xlsx`, writes `data/predictions/mlp_predictions.xlsx`, and prints MAE/RMSE per axis.

### 4. Plotting — `src/plotting/plot_results.py`
Generates the scatter, 3D trajectory, error distribution, and error-over-time plots used in the paper. Outputs to `data/plots/`.

## Reproducing Individual Paper Figures

Each `making_figures/FigureX_*` folder is self-contained with its own data, model, scalers, and scripts. To regenerate a figure:

```bash
cd making_figures/Figure3_3D_Trajectory
./run.sh
```

`run.sh` runs `predict_and_export.py` followed by `plot_results.py` and writes the figure image into the same folder. Figures 1 and 2 are static images (drone photo and EPM close-up).

## Installation

```bash
git clone https://github.com/ARTS-Laboratory/Paper-2026-MLPUAV-SPIE.git
cd Paper-2026-MLPUAV-SPIE
pip install -r requirements.txt
```

## Running the Full Pipeline

```bash
python src/preprocessing/align_and_format.py
python src/modeling/train_model.py
python src/modeling/predict_and_export.py
python src/plotting/plot_results.py
```

## Notes

- All coordinates are in millimeters.
- Raw video capture and YOLO labeling are outside this repo; the workflow starts from the provided Excel files.
