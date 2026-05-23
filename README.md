# Deep Learning-Enabled Phase Aberration Correction for Enhanced Ultrasound Imaging

This repository contains the simulation framework, deep learning models, and evaluation tools for my Bachelor's Thesis project at the University of Tehran, Faculty of Mechanical Engineering. The project focuses on correcting phase aberrations caused by tissue heterogeneity in breast cancer ultrasound imaging using deep neural networks.

## 📋 Project Overview

In diagnostic ultrasound, conventional image reconstruction algorithms assume a homogeneous medium with a constant speed of sound ($1540\text{ m/s}$). However, biological tissues are highly heterogeneous. Variations in the density, elasticity, and spatial coordinates of skin layers (epidermis, dermis), connective tissues, and fatty inclusions alter the local acoustic impedance and speed of sound. This mismatch leads to distortions in the acoustic wavefront—known as **Phase Aberration**—which degrades focus, reduces image resolution, and diminishes contrast in breast mammograms.

This project introduces an intelligent framework that bridges deep learning with classical acoustic signal processing. Instead of computationally expensive tomographic speed-of-sound reconstruction or error-prone adjacent-element correlation methods, a Deep Neural Network (DNN) is trained to map pre-processed raw RF (Radio Frequency) data directly to a zero-mean relative channel time-delay vector ($\Delta\tau$). These predicted delays are then applied channel-wise to realign the phase before standard Delay-and-Sum (DAS) beamforming, enabling real-time, high-resolution, and robust clinical ultrasound imaging.

---

## 🛠️ System Architecture & Workflow

The phase aberration correction pipeline consists of three main operational phases:

```
[ PHYSICS ENGINE: k-Wave ] 
       │ (Simulate Heterogeneous & Homogeneous Media)
       ▼
[ RAW RF SIGNALS ] ───► [ GROUND TRUTH CALCULATION ]
       │                      │ (Cross-Correlation)
       ▼                      ▼
[ PRE-PROCESSING ]      [ ZERO-MEAN RELATIVE DELAYS ] (Δτ [64×1])
(DC Removal, Norm)            │
       │                      │
       ▼                      ▼
[ DEEP NEURAL NETWORK ] ──► [ LOSS FUNCTION ] (MSE / MAE)
       │ (Predicts Delays)
       ▼
[ APPLY CORRECTIVE DELAYS ] (Per-channel shift)
       │
       ▼
[ BEAMFORMER & IMAGING ] (DAS -> Envelope Detection -> Log Compression)
       │
       ▼
[ ABERRATION-CORRECTED 2D IMAGE ]
```

1. **Physical Wave Propagation Modeling:** 2D ultrasonic wave propagation is simulated using the pseudospectral `k-Wave` MATLAB/Python toolbox. A linear array configuration (64 to 192 channels) operating at central frequencies of $5.0\text{ to }7.5\text{ MHz}$ transmits plane waves into a multi-layered breast tissue model.
2. **RF Signal Pre-Processing:** Raw RF matrices ($X \in \mathbb{R}^{N_{ch} \times N_t}$) undergo time-window cropping, DC offset removal, and min-max normalization to isolate meaningful wavefront curvatures.
3. **Deep Learning Reference Mapping:** The model learns a direct mapping function:
   $$\mathcal{F}: \mathbf{X}_{hetero} \longrightarrow \Delta\tau$$
4. **Beamforming & Reconstruction:** Corrected delays are fed into a Delay-and-Sum beamformer, followed by analytic envelope detection and logarithmic dynamic range compression to construct the final 2D B-mode image.

---

## 🔬 Physics Simulation & Dataset Generation

The training and validation dataset is synthetically derived using **k-Wave** to ensure exact ground-truth control:
* **Tissue Characterization:** Built based on anatomical and dermatological references (including multi-layered skin structures like the epidermis and dermis) and complex breast fatty distributions.
* **Acoustic Variability:** Medium maps account for localized speed-of-sound variations ($1400\text{ m/s}$ to $1700\text{ m/s}$), structural refraction, and complex multi-target diffractions.
* **Ground Truth Labeling:** Calculated via cross-correlation between the backscattered signals of the heterogeneous media ($R$) and an identical reference homogeneous media ($R_0$) across each channel:
  $$\Delta\tau_{raw} = \arg\max_{\tau} \mathcal{R}_{R, R_0}(\tau)$$
  Labels are subsequently centered to a zero-mean relative delay vector to isolate pure geometric and aberrator phase perturbations.

---

## 📂 Repository Structure

```
├── simulation/
│   ├── models/                # Breast tissue phantoms & sound speed maps
│   ├── kwave_generate_rf.m    # k-Wave simulation execution script
│   └── extract_labels.py      # Cross-correlation and zero-mean delay labeller
├── src/
│   ├── preprocessing.py       # RF matrix windowing, normalization, DC removal
│   ├── networks.py            # Deep Neural Network topologies (CNN/ResNet backbones)
│   ├── beamformer.py          # Delay-and-Sum (DAS) beamforming module
│   └── train.py               # Model optimization loop and learning rate schedules
├── evaluation/
│   ├── metrics.py             # MAE, RMSE, and cross-correlation tracking
│   └── image_quality.py       # Contrast-to-Noise Ratio (CNR) and FWHM calculators
├── docs/
│   └── Proposal.pdf           # Approved undergraduate project proposal
├── requirements.txt           # Python dependency manifest
└── README.md                  # Project documentation
```

---

## 📈 Evaluation Metrics

The performance and clinical reliability of the intelligent framework are quantified across two dimensions:

### 1. Delay Estimation Accuracy
* **Mean Absolute Error (MAE):** Measures average absolute residual delay deviations.
* **Root Mean Squared Error (RMSE):** Penalizes larger outlier delay mispredictions.
* **Correlation Coefficient ($R$):** Evaluates phase profile synchronization against cross-correlation benchmarks.

### 2. Reconstructed Image Quality
* **Spatial Resolution:** Quantified via the Full Width at Half Maximum (FWHM) of point targets.
* **Contrast-to-Noise Ratio (CNR):** Evaluates target visibility and boundary sharpness of cystic or solid tumor masses.
* **Robustness Metrics:** Validated under additive white Gaussian noise (AWGN), variable channel gain fluctuations, and random element-masking to replicate broken or dead transducer elements.

---

## 🗓️ Project Timeline

The project timeline spans across the academic year 1404–1405 (Iranian Calendar):
* **Jul – Aug 2025:** Literature review on tissue heterogeneity models.
* **Aug – Oct 2025:** Setup of `k-Wave` 2D ultrasound domain parameters.
* **Oct – Nov 2025:** Large-scale RF dataset wave-propagation simulations.
* **Nov – Dec 2025:** Cross-correlation label extraction and zero-mean indexing.
* **Dec 2025 – Feb 2026:** Pre-processing pipeline development and DNN design/training.
* **Feb – Jun 2026:** Robustness testing, image quality assessment, and thesis drafting.

---

## 🎓 Academic Credit & Supervision

* **Author:** Seyedali Ameli (Alirad Ameli)
* **Affiliation:** University of Tehran, Faculty of Mechanical Engineering, Perdis of Engineering Faculties
* **Supervisor:** Dr. Mohammad Hossein Amini (Assistant Professor)

---

## 📚 References


1. R. Ali et al., "Aberration correction in diagnostic ultrasound: A review of the prior field and current directions," *Ultrasound in Medicine & Biology*, vol. 49, no. 11, 2023.
2. R. S. C. Cobbold, *Foundations of Biomedical Ultrasound*, Oxford University Press, 2006.
3. M. Tabei, T. D. Mast, and R. C. Waag, "Simulation of ultrasonic focus aberration and correction through human tissue," *J. Acoust. Soc. Am.*, vol. 113, no. 2, 2003.
4. C. E. M. Griffiths et al., *Rook's Textbook of Dermatology*, 10th ed., Wiley-Blackwell, 2024.
5. F. Khun Jush et al., "Deep learning for ultrasound speed-of-sound reconstruction: Impacts of training data diversity on stability and robustness," *MELBA*, vol. 2, 2023.
6. J. F. Havlice and J. C. Taenzer, "Medical ultrasonic imaging: An overview of principles and instrumentation," *Proc. IEEE*, vol. 67, no. 4, 1979.
7. G. C. Ng et al., "A comparative evaluation of several algorithms for phase aberration correction," *IEEE Trans. UFFC*, vol. 41, no. 5, 1994.
8. 
