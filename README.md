# Anomaly Detection Model for SWaT Dataset

A multi-stage deep learning system for detecting cyberattacks in industrial control systems (ICS/SCADA), specifically designed for the **SWaT (Secure Water Treatment) 2019 dataset**.

## Overview

This project implements an anomaly/cyberattack detection pipeline using CNN-LSTM models trained independently for each physical stage of a water treatment plant. The system detects attacks by computing prediction errors, converting them to Z-scores, and applying sliding-window thresholding.

## Performance

Precision : 0.8457
Recall    : 1.0000
F1-Score  : 0.9164
## Architecture

### Pipeline
1. **Data Ingestion** - Loads SWaT sensor data (77 features across 6 stages: P1-P6)
2. **Preprocessing** - Handles timestamps, attack labeling, data cleaning, and feature selection
3. **Feature Engineering** - Adds derivative features (first-order differences)
4. **Sequence Creation** - Sliding windows of 200 time steps
5. **Model Training** - 1D-CNN for the whole system 
6. **Anomaly Detection** - MSE errors → Z-scores → sliding-window thresholding
7. **Ensemble** - Logical OR combination of all stage predictions

### Model Architecture (whole system)
- **Input**: shape `(time_steps, n_features_in)`
- **Conv1D**: 64 filters, kernel size 3, padding 'same', ReLU activation
- **BatchNormalization**
- **MaxPooling1D**: pool size 2
- **Conv1D**: 128 filters, kernel size 3, padding 'same', ReLU activation
- **BatchNormalization**
- **MaxPooling1D**: pool size 2
- **Conv1D**: 256 filters, kernel size 3, padding 'same', ReLU activation
- **BatchNormalization**
- **MaxPooling1D**: pool size 2
- **Flatten**
- **Dense**: 128 units, ReLU activation
- **Dropout**: 0.4
- **Dense**: `n_features_out` units
- **Optimizer**: Adam, **Loss**: MSE

## Detected Attack Types

The model was evaluated against 6 known attack types in the SWaT 2019 dataset:

1. **Attack 1** - Manipulation of LIT101 (level indicator)
2. **Attack 2** - Manipulation of MV201/P101 (valve/pump)
3. **Attack 3** - Manipulation of FIT401 (flow indicator)
4. **Attack 4** - Manipulation of LIT301 (level indicator)
5. **Attack 5** - Manipulation of P501 (pump)
6. **Attack 6** - Multi-point manipulation

## Requirements

- Python 3.12+
- TensorFlow
- Pandas
- NumPy
- scikit-learn
- Matplotlib
- Seaborn

## Usage

### Running on Kaggle (Recommended)

1. Upload `data2.ipynb` to Kaggle
2. Attach the SWaT dataset (Kaggle dataset ID: 8365136, version ID: 13891745)
3. Update the data path in the notebook to match your Kaggle input location
4. Run all cells sequentially (~19 minutes on Kaggle CPU)

### Running Locally

```bash
# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install tensorflow pandas numpy scikit-learn matplotlib seaborn

# Place SWaT.csv in the expected path and run the notebook
jupyter notebook data2.ipynb
```

## Project Structure

```
.
├── data2.ipynb          # Main notebook with complete pipeline
├── .venv/               # Python virtual environment
└── README.md            # This file
```

## Key Design Decisions

- **Multi-stage architecture**: Independent models per physical stage enable localized anomaly detection
- **Derivative features**: First-order differences capture rate-of-change information, doubling input features
- **Strict anomaly condition**: Uses "all values in window exceed threshold" rule (product operator)
- **Recovery period handling**: Extends attack labels by 600 seconds post-attack to account for system recovery
- **Grid search**: Automatically searches optimal threshold (T) and window size (W) per stage

## Dataset

- **Name**: SWaT (Secure Water Treatment) 2019
- **Source**: iTrust, Singapore University of Technology and Design
- **Features**: 77 sensor readings across 6 physical stages
- **Train/Test Split**: Time-based at 2019-07-20 07:00:00 UTC
  - Train: 8,996 samples (all normal)
  - Test: 6,000 samples (4,019 normal + 1,981 attack)

## Repository

Source code: [https://github.com/NguyenHa554/Anomaly-detection-model](https://github.com/NguyenHa554/Anomaly-detection-model)
