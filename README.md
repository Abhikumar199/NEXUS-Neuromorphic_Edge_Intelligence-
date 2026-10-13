# NEXUS: Neuromorphic Edge Intelligence for Sewer Anomaly Detection

**An event-driven Spiking Neural Network (SNN) framework for detecting sewer pipeline anomalies at the edge.**

[![Platform](https://img.shields.io/badge/Platform-ESP32-E7352C?logo=espressif&logoColor=white)](https://www.espressif.com/)
[![Framework](https://img.shields.io/badge/AI-Spiking%20Neural%20Networks-6C5CE7)](https://en.wikipedia.org/wiki/Spiking_neural_network)
[![Application](https://img.shields.io/badge/Application-Urban%20Drainage-2D8C7F)](#)
[![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)](#)

NEXUS is a research prototype for **real-time sewer pipeline monitoring** using low-cost water-quality and flow sensors, spike-based inference, and local processing on an ESP32 microcontroller. The goal is to identify changes associated with contamination and blockage early, while limiting dependence on cloud processing.

> **Research result:** The accompanying study reports **86.60% accuracy**, **86.40% precision**, **86.60% recall**, and an **86.43% F1-score** on a collected dataset of approximately **3,545 samples**. These are experimental results from a controlled PVC-pipe testbed, not a claim of field validation across municipal sewer networks.

## Contents

- [Overview](#overview)
- [System architecture](#system-architecture)
- [Detection classes](#detection-classes)
- [Hardware](#hardware)
- [Dataset](#dataset)
- [SNN approach](#snn-approach)
- [Results](#results)
- [Repository structure](#repository-structure)
- [Getting started](#getting-started)
- [Limitations and future work](#limitations-and-future-work)
- [Research context](#research-context)
- [Acknowledgements](#acknowledgements)

## Overview

Conventional monitoring systems can depend on periodic inspection, continuous sampling, and remote processing. NEXUS explores a different design: sensor measurements are transformed into compact features and spike trains, then processed locally by a lightweight SNN.

The prototype combines:
- **Multi-sensor monitoring:** total dissolved solids (TDS), turbidity, and water flow.
- **Feature engineering:** smoothed sensor signals and flow-derived indicators.
- **Spike-based inference:** rate-based encoding and Leaky Integrate-and-Fire (LIF) neurons.
- **Edge deployment:** ESP32-based data acquisition and inference firmware.
- **Local alerting concept:** Wi-Fi notifications for conditions classified as anomalous.

The current setup is a **hybrid sensing and event-driven inference system**: sensor values are sampled at intervals, while spike-based inference represents the experimental neuromorphic component. It is not a dedicated neuromorphic chip implementation.

## System architecture

```text
TDS sensor ─────┐
Turbidity sensor ├──> ESP32 data acquisition
Flow sensor ────┘              │
                               ▼
                    Smoothing / filtering
                               │
                               ▼
                   Feature extraction (6)
                               │
                               ▼
                    Rate-based spike encoding
                               │
                               ▼
                      LIF-based SNN
                               │
                               ▼
                   Four-class prediction
                               │
                 ┌─────────────┴─────────────┐
                 ▼                           ▼
          Normal monitoring          Anomaly / alert path
```

The training workflow is performed offline in notebooks. The repository contains saved model parameters and an Arduino sketch; check the implementation details before assuming the firmware exactly matches the six-feature SNN described in the study.

## Detection classes

| Label | Condition | Experimental interpretation |
|---|---|---|
| 0 | Normal flow | Clean-water baseline |
| 1 | Contaminated flow | Water with salts and soap |
| 2 | Muddy water / partial blockage | Increased suspended particles and reduced-flow scenario |
| 3 | Severe blockage | Strongly reduced or obstructed flow scenario |

These classes describe controlled test conditions and should not be interpreted as a validated diagnosis for every real-world sewer condition.

## Hardware

| Component | Model / type | Role |
|---|---|---|
| Microcontroller | ESP32 (ESP-WROOM-32) | Sensor acquisition and edge processing |
| TDS sensor | Analog TDS sensor | Dissolved-solids proxy |
| Turbidity sensor | Water turbidity sensor | Suspended-particle measurement |
| Flow sensor | YF-S201 | Flow-rate measurement |
| Testbed | PVC pipe setup | Controlled flow experiments |
| Power components | Li-ion battery, TP4056, MT3608 | Portable power and voltage regulation |
| Enclosure | Waterproof case (prototype design) | Protection of electronics |

**Safety note:** This prototype is for controlled experimentation. It is not certified for deployment in hazardous wastewater environments. Electrical isolation, water ingress protection, sensor calibration, and safe maintenance practices require further validation.

## Dataset

The study reports approximately **3,545 samples** collected under controlled testbed conditions. The feature vector used in the described SNN consists of six variables:

1. Smoothed TDS
2. Smoothed turbidity
3. Flow rate
4. Flow delta
5. Low-flow indicator
6. Flow ratio

Reported class distribution:

| Label | Class | Samples |
|---:|---|---:|
| 0 | Normal clean-water flow | 972 |
| 1 | Contaminated flow (salts and soap) | 1,095 |
| 2 | Muddy water / partial blockage | 863 |
| 3 | Severe blockage condition | 615 |
| | **Total** | **3,545** |

The repository includes raw sensor logs, a preprocessing notebook, and a processed CSV. Please review the notebooks for the precise filtering, splitting, normalization, and label-generation steps used in a given experiment.

## SNN approach

The study describes a compact feed-forward SNN using Leaky Integrate-and-Fire neurons.

| Setting | Description |
|---|---|
| Input features | 6 |
| Hidden layer | 20 spiking neurons |
| Output layer | 4 classes |
| Encoding | Rate-based Bernoulli spike encoding |
| Simulation steps | 40 |
| Neuron model | Leaky Integrate-and-Fire (LIF) |
| Training | Offline; Adam optimizer and cross-entropy objective are described in the paper |
| Deployment target | ESP32 microcontroller |

For normalized feature (x_i \in [0,1]), rate-based encoding generates binary spikes over a fixed time window, with larger feature values corresponding to a greater probability of spiking. The network integrates incoming spikes through membrane dynamics and predicts a class from accumulated output activity.

**Implementation note:** The checked-in Arduino sketch currently defines a **3–16–8–4 ANN-style forward pass** and uses placeholder input values. Therefore, it should not be treated as proof that the six-input, 20-neuron SNN described above is fully reproduced in the current firmware. The notebooks and embedded model files should be inspected together when reproducing results.

## Results

The accompanying paper reports the following aggregate metrics for the SNN experiment:

| Metric | Reported result |
|---|---:|
| Accuracy | **86.60%** |
| Precision | **86.40%** |
| Recall | **86.60%** |
| F1-score | **86.43%** |
| Dataset size | ~3,545 samples |
| ESP32 current reported in the study | ~48–52 mA for the SNN setup |

The repository's `results/` directory contains evaluation figures, including ROC curves, a precision–recall curve, a confusion matrix, class-wise metrics, training curves, and a spike-activity visualization.

### Evaluation plots

**ROC curves**

![ROC curves](results/AUC-ROC.png)

**Precision–recall curves**

![Precision-recall curves](results/precision_recall_curve.png)

**Confusion matrix**

![Confusion matrix](results/Confusion_matrix.png)

**Class-wise performance**

![Class-wise performance](results/Class%20wise%20matrix.png)

**Training accuracy**

![Training accuracy](results/Accuracy%20curve.png)

**Training loss**

![Training loss](results/training_loss.png)

**Spike activity**

![Spike activity](results/spike%20activity.png)

### Interpreting the power measurements

The paper reports approximate current ranges of **50–60 mA for the ANN baseline** and **48–52 mA for the SNN setup**, and discusses potential long-term benefits from sparse/event-triggered processing. These current ranges alone do not establish a complete energy-efficiency advantage. A controlled comparison should measure the same hardware configuration, peripherals, operating voltage, duty cycle, and workload, and report energy over a defined duration.

## Repository structure

```text
.
├── Arduino_IDE/
│   └── Arduino_IDE.ino          # ESP32 / Arduino firmware
├── Training/
│   ├── ann_training.ipynb       # ANN baseline training notebook
│   ├── snn_training.ipynb       # SNN training notebook
│   └── final_data.csv           # Processed dataset
├── raw_data/
│   ├── normal_baseline.txt      # Baseline sensor log
│   ├── final_bloackage.txt      # Blockage-related sensor log
│   ├── muddy+soap_anomly.txt    # Muddy/soap anomaly log
│   ├── soap_round_final.txt     # Sensor log
│   ├── soap_round_light.txt     # Sensor log
│   └── preprocessing.ipynb     # Data preprocessing workflow
├── results/
│   ├── AUC-ROC.png
│   ├── Accuracy curve.png
│   ├── Class wise matrix.png
│   ├── Confusion_matrix.png
│   ├── precision_recall_curve.png
│   ├── spike activity.png
│   └── training_loss.png
├── fc1_w.npy / fc1_b.npy        # Saved parameters
├── fc2_w.npy / fc2_b.npy        # Saved parameters
├── python server.py             # Basic TCP receiver prototype
└── README.md
```

## Getting started

### 1. Clone the repository

```bash
git clone https://github.com/Abhikumar199/NEXUS-Neuromorphic_Edge_Intelligence-.git
cd NEXUS-Neuromorphic_Edge_Intelligence-
```

### 2. Explore the data and training workflow

Open the notebooks in Jupyter Notebook or JupyterLab:

- `raw_data/preprocessing.ipynb` — review the raw sensor data and preprocessing.
- `Training/ann_training.ipynb` — inspect the ANN baseline.
- `Training/snn_training.ipynb` — inspect the SNN experiment.

The notebooks may require Python packages such as NumPy, pandas, Matplotlib, scikit-learn, and the SNN framework used in the notebook. Because a pinned environment file is not currently provided, install dependencies according to the imports in the notebook and record the versions for reproducible experiments.

### 3. Inspect the ESP32 code

Open `Arduino_IDE/Arduino_IDE.ino` in the Arduino IDE and select the appropriate ESP32 board and serial port. Verify the firmware's input features, sensor pins, model architecture, and weight mapping before connecting hardware. The currently checked-in sketch contains placeholder inputs; it is not a ready-to-run end-to-end sensor-to-SNN deployment.

### 4. Optional: run the TCP receiver prototype

The file `python server.py` opens a basic TCP server on port `5000` and prints received messages. It is a minimal receiver for experimentation, not a complete dashboard or alerting service. Run it only in a trusted environment and restrict network access as appropriate.

## Limitations and future work

- **Controlled testbed:** Results come from a small PVC-pipe prototype; validation on real sewer infrastructure remains future work.
- **Sensor reliability:** Distinguishing partial from severe blockage is difficult when flow approaches zero or sensor readings overlap.
- **Firmware consistency:** The checked-in Arduino sketch and the paper's SNN architecture are not yet fully aligned.
- **Energy evaluation:** More rigorous measurements of energy per inference, duty-cycle behavior, and communication overhead are needed.
- **Robustness:** Broader experiments should evaluate changing flow rates, sensor drift, varied contaminant concentrations, and environmental noise.
- **Future directions:** The study proposes evaluating additional audio and pressure sensors, adaptive thresholds, hybrid SNN approaches, and neuromorphic hardware.

## Research context

**Project title:** *NEXUS: An Event-Driven Neuromorphic Edge Intelligence Framework for Real-Time Urban Sewer Blockage Detection*

The project combines low-cost sensing, edge AI, and spiking neural networks to explore proactive infrastructure monitoring.

## Acknowledgements

The accompanying study acknowledges support from the Atal Bihari Vajpayee–Indian Institute of Information Technology and Management, Gwalior, under the Faculty Initiation Grant (FIG) scheme, Grant No. **ABV-IIITM/DoRC/FIG/2025-26/2379**.

## Citation

If you use this work, please cite the project and its associated paper. Add the final publication venue, year, DOI, and author-approved citation once those details are available.

---

**Maintainer:** [Abhishek Kumar](https://github.com/Abhikumar199)  
**Repository:** [NEXUS-Neuromorphic Edge Intelligence](https://github.com/Abhikumar199/NEXUS-Neuromorphic_Edge_Intelligence-)
