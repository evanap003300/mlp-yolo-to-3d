# UAV localization from stereo camera detections

Code from my work on *Machine learning-based UAV localization for
under-structure sensor deployment in GPS-denied environments* (SPIE Smart
Structures + NDE 2026), done in the ARTS Lab at the University of South
Carolina.

GPS does not work under a bridge, which is where these drones place sensors.
Instead, two fixed cameras watch the drone, YOLO finds it in each frame, and a
model maps those pixel positions to real 3D coordinates in millimeters. An
OptiTrack motion-capture system provides the ground truth to train against.

This repo holds the first model I built for that mapping, a small MLP, with its
preprocessing, trained weights and figure scripts. The paper goes on to compare
four classical regressors on the same mapping (polynomial, support vector,
random forest, gradient boosting). SVR with an RBF kernel came out best, at
16.48 mm RMSE under 5-fold cross-validation.

## Paper

Md Asifuzzaman Khan, Joud N. Satme, Mark Zheng, Samuel Tadamatla, Evan Phillips,
Korebami Adebajo, Nolan Shute, Ryan Yount, and Austin R. J. Downey. "Machine
learning-based UAV localization for under-structure sensor deployment in
GPS-denied environments." *Sensors and Smart Structures Technologies for Civil,
Mechanical, and Aerospace Systems 2026*, SPIE, April 2026.
[doi:10.1117/12.3089966](https://doi.org/10.1117/12.3089966)

A copy is in this repo at [paper/MLPUAV-SPIE.pdf](paper/MLPUAV-SPIE.pdf).

## Project Structure

```
mlp-yolo-to-3d/
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
├── making_figures/     # Self-contained folders, one per figure of the MLP results
│   ├── Figure1_Drone/
│   ├── Figure2_EPM/
│   ├── Figure3_3D_Trajectory/
│   ├── Figure4_Error_Distribution/
│   ├── Figure5_Scatter_Plots/
│   └── Figure6_Error_Over_Time/
├── paper/              # The published paper (PDF)
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
Generates the scatter, 3D trajectory, error distribution, and error-over-time plots for the MLP. Outputs to `data/plots/`.

## Reproducing Individual Figures

Each `making_figures/FigureX_*` folder is self-contained with its own data, model, scalers, and scripts. To regenerate a figure:

```bash
cd making_figures/Figure3_3D_Trajectory
./run.sh
```

`run.sh` runs `predict_and_export.py` followed by `plot_results.py` and writes the figure image into the same folder. Figures 1 and 2 are static images (drone photo and EPM close-up).

## Installation

```bash
git clone https://github.com/evanap003300/mlp-yolo-to-3d.git
cd mlp-yolo-to-3d
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
