# EEG Emotion Detection

Binary emotion classification (positive vs negative valence) from EEG signals using the OpenNeuro dataset [ds003004](https://openneuro.org/datasets/ds003004).

---

## Overview

This project processes raw EEG data from 34 subjects who underwent guided emotional imagery, and trains a classifier to distinguish positive from negative emotional states using brain activity alone.

Each subject listened to 15 emotion-induction narratives and pressed a sensor when they began feeling the suggested emotion. The EEG signal at that moment was used as the ground truth label.

---

## Dataset

**OpenNeuro ds003004 — Imagined Emotion**

- 34 subjects
- 224 EEG channels (high-density cap)
- 15 emotion categories per subject: anger, awe, compassion, content, disgust, excite, fear, frustration, grief, happy, jealousy, joy, love, relief, sad
- ~75 minutes of recording per subject
- Sampling rate: 256 Hz

Download a single subject to get started:

```bash
python -m openneuro download --dataset ds003004 --target-dir ./ds003004 --include sub-01
```

Download all subjects:

```bash
python -m openneuro download --dataset ds003004 --target-dir ./ds003004
```

---

## Emotion Labels

The 15 emotions are grouped into binary valence classes:

| Class | Emotions |
|---|---|
| **Positive (1)** | awe, compassion, content, excite, happy, joy, love, relief |
| **Negative (0)** | anger, disgust, fear, frustration, grief, jealousy, sad |

---

## Pipeline

```
Raw EEG (.set)
    │
    ├── 1. Filtering          high-pass 1 Hz, low-pass 128.00 Hz, notch 50 Hz
    ├── 2. Bad channel detection   statistical outlier detection → interpolation
    ├── 3. ICA                     remove eye blink and eye movement components
    ├── 4. Re-referencing          average reference across all 224 channels
    ├── 5. Event extraction        parse emotion onset annotations
    ├── 6. Epoching                60-second windows per emotion trial
    ├── 7. Feature extraction      band power (delta/theta/alpha/beta/gamma) per channel
    └── 8. Classification          SVM with leave-one-subject-out cross-validation
```

---

## Feature Extraction

For each 60-second epoch, Welch's power spectral density is computed and averaged within five canonical frequency bands across all 224 channels:

| Band | Range | Emotion relevance |
|---|---|---|
| Delta | 1–4 Hz | Baseline, slow cortical activity |
| Theta | 4–8 Hz | Anxiety, fear, frustration |
| Alpha | 8–13 Hz | Relaxation vs emotional arousal |
| Beta | 13–30 Hz | Active arousal, stress, excitement |
| Gamma | 30–45 Hz | Intense emotional processing |

This produces a **1120-dimensional feature vector** (224 channels × 5 bands) per epoch, log-transformed for normality.

---

## Requirements

```bash
pip install mne numpy scikit-learn matplotlib pyprep openneuro-py
```

| Package | Version |
|---|---|
| Python | 3.11+ |
| MNE | 1.6+ |
| scikit-learn | 1.4+ |
| NumPy | 1.26+ |

---

## Usage

Open `index.ipynb` and run cells in order. Update the dataset path in the loading cell to point to your local copy of ds003004:

```python
sub1_path = 'ds003004/sub-01/eeg/sub-01_task-ImaginedEmotion_eeg.set'
```

---

## Notebook Structure

| Section | Description |
|---|---|
| **Data loading** | Load raw `.set` file using MNE-EEGLAB reader |
| **Preprocessing** | Filter, detect and interpolate bad channels |
| **ICA** | Fit ICA, identify and remove ocular artifacts |
| **Re-referencing** | Apply average reference |
| **Event extraction** | Parse annotations, map to emotion labels |
| **Epoching** | Cut continuous data into 60s emotion segments |
| **Feature extraction** | Compute log band power per channel per band |
| **Classification** | Train SVM, evaluate with cross-validation |

---

## Results

Single subject (sub-01) pilot result:

- 9 epochs survived rejection (out of 15)
- 5-fold cross-validation balanced accuracy: ~55%
- Note: single-subject results are not meaningful due to very few trials

Cross-subject results (all 34 subjects pooled, leave-one-subject-out):

- To be updated after full dataset run

---

## References

- [OpenNeuro ds003004](https://openneuro.org/datasets/ds003004)
- [MNE-Python](https://mne.tools/stable/index.html)
- [MNE Filtering](https://mne.tools/stable/generated/mne.io.Raw.html#mne.io.Raw.filter)
- [MNE ICA](https://mne.tools/stable/generated/mne.preprocessing.ICA.html)
- [PyPREP Bad Channel Detection](https://pyprep.readthedocs.io/en/latest/generated/pyprep.NoisyChannels.html)
- [MNE Interpolate Bads](https://mne.tools/stable/generated/mne.io.Raw.html#mne.io.Raw.interpolate_bads)
