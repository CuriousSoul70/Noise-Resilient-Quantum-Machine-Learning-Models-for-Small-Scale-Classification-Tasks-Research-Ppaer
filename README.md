# Noise-Resilient Quantum Machine Learning Models for Small-Scale Classification Tasks

A research project investigating how quantum noise affects Quantum Kernel Support Vector Machine (QKSVM) classifiers built on parameterized quantum circuits, and evaluating strategies to improve their resilience on NISQ-era hardware.

---

## Overview

Quantum Machine Learning (QML) models running on Noisy Intermediate-Scale Quantum (NISQ) devices suffer significant performance degradation due to hardware noise. This project systematically studies that degradation and proposes a novel evaluation metric — the **Quantum Robustness Score (QRS)** — to quantify and compare noise resilience across different circuit configurations, noise models, and benchmark datasets.

Three circuit configurations are compared:
- **Baseline** — ZZFeatureMap with depth 2
- **Shallow** — ZZFeatureMap with depth 1
- **Noise-Aware** — depth 2, trained on a noisy kernel

Three noise models are evaluated:
- Depolarizing noise
- Bit-flip noise
- Phase damping noise

All experiments are fully simulation-based using Qiskit 2.x — no quantum hardware access required.

---

## Novel Contribution

This work introduces the **Quantum Robustness Score (QRS)**:

```
QRS = (1/K) × Σ [ Accuracy(p_k) / Accuracy(p=0) ]
```

QRS measures the average fraction of noiseless performance a model retains as noise increases. Unlike raw accuracy at a single noise level, QRS normalizes by baseline performance — making it comparable across datasets and configurations with different starting accuracies.

---

## Repository Structure

```
├── utils.py              # Dataset loading, quantum kernel, noise application, QRS metric
├── experiment.py         # Main experiment loop — runs all 450 experiments, saves results.pkl
├── plot_results.py       # Generates all 4 figures from results.pkl
├── requirements.txt      # Python dependencies
└── README.md
```

---

## Datasets

| Dataset | Samples | Features (after PCA) | Task |
|---|---|---|---|
| Iris | 100 | 4 | Binary (class 0 vs 1) |
| Breast Cancer Wisconsin | 569 | 4 | Binary (malignant vs benign) |
| Wine | 130 | 4 | Binary (class 0 vs 1) |

All features are scaled to [0, π] for quantum angle encoding. Dimensionality reduction via PCA where needed.

---

## Experimental Design

| Parameter | Values |
|---|---|
| Noise types | Depolarizing, Bit-Flip, Phase Damping |
| Noise levels | 0.01, 0.05, 0.10 |
| Random seeds | 0, 1, 2, 3, 4 |
| Total experiment instances | 450 (+ 45 noiseless baselines) |
| Metric | QRS (primary), Test Accuracy (secondary) |

---

## Setup and Usage

**Step 1 — Clone the repository**
```bash
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name
```

**Step 2 — Install dependencies**
```bash
pip install -r requirements.txt
```

**Step 3 — Run experiments**
```bash
python experiment.py
```
This will print result tables to the console and save `results.pkl` in the same folder. Takes approximately 1–3 minutes.

**Step 4 — Generate figures**
```bash
python plot_results.py
```
This reads `results.pkl` and saves four PNG figures in the same folder.

---

## Results

### Noiseless Accuracy (Mean ± Std, 5 seeds)

| Configuration | Iris | Breast Cancer | Wine |
|---|---|---|---|
| Baseline (depth=2) | 0.792 ± 0.064 | 0.929 ± 0.016 | 0.661 ± 0.107 |
| Shallow (depth=1) | **0.952 ± 0.030** | **0.934 ± 0.011** | **0.776 ± 0.031** |
| Noise-Aware (depth=2) | 0.784 ± 0.060 | 0.910 ± 0.010 | 0.612 ± 0.098 |

### Average QRS

| Configuration | Iris | Breast Cancer | Wine |
|---|---|---|---|
| Baseline (depth=2) | 0.918 | 0.643 | 0.888 |
| Shallow (depth=1) | 0.915 | **0.812** | **0.941** |
| Noise-Aware (depth=2) | 0.913 | 0.757 | 0.922 |

### Best Configuration Per Noise Type

| Dataset | Depolarizing | Bit-Flip | Phase Damping |
|---|---|---|---|
| Iris | Shallow | Baseline | Shallow |
| Breast Cancer | Shallow | Shallow | Shallow |
| Wine | Shallow | Noise-Aware | Shallow |

---

## Key Findings

1. **Shallow circuits win on both accuracy and robustness.** The depth=1 configuration achieves the highest noiseless accuracy on all three datasets and the highest average QRS on two of three datasets. Reducing circuit depth limits noise accumulation without sacrificing classification performance on small-scale tasks.

2. **Bit-flip noise is the most destructive.** Across all configurations and datasets, bit-flip noise produces the lowest QRS values — identifying it as the primary noise threat for NISQ quantum classifiers.

3. **Noise-aware training provides selective advantage.** It outperforms both other strategies specifically under bit-flip noise on the Wine dataset, suggesting it is complementary to shallow circuit design rather than a universal replacement.

4. **QRS reveals what accuracy alone cannot.** The Baseline configuration achieves 0.929 noiseless accuracy on Breast Cancer but a QRS of only 0.643 — losing 36% of its performance under noise. The Shallow configuration achieves nearly identical noiseless accuracy (0.934) but a QRS of 0.812. Without QRS, these two would appear equivalent.

---

## Dependencies

```
qiskit>=2.0.0
qiskit-aer>=0.15.0
qiskit-machine-learning>=0.8.0
scikit-learn>=1.3.0
numpy>=1.24.0
matplotlib>=3.7.0
```

---

## Note on Noise Modeling

Noise is applied analytically to the ideal quantum kernel matrix using the depolarizing channel formulation from quantum information theory:

```
K_noisy = decay × K_ideal + (1 − decay) × (1 / 2^n)
```

This approach is mathematically equivalent to running a full noisy circuit simulator and is standard practice in theoretical QML research. Finite-shot measurement noise (256 shots per kernel entry) is also included using a Binomial variance model.

---

## Research Context

This project is part of the K1000 Research Program. The work is simulation-based and designed to be fully reproducible on a standard laptop without quantum hardware access.

---

## License

This project is for academic research purposes.
