# Unknown-Attack Screening in Network Flow Data Using Open-Set Learning

Applied Machine Learning (Summer 2026)

**Team:** Dhruv Sanjay Patel, Nehal Shivaram
**Team ID:** ML_DS_120A_16 · Project 6

---

## Problem Statement

Traditional network intrusion detection systems operate under a **closed-set assumption** — they can only recognize attack types they were explicitly trained on, which means they struggle to flag novel or modified exploits.

This project builds an **Open-Set Learning** framework for network intrusion detection that:
- Classifies known traffic behaviors (benign vs. known attacks), and
- Flags traffic that doesn't resemble anything seen during training as **"Suspicious/Unknown"**,

without ever exposing the model to the hidden attack family during training — simulating a real zero-day scenario.

## Dataset

- **Source:** [CICIDS2017](https://www.unb.ca/cic/datasets/ids-2017.html), Canadian Institute for Cybersecurity (UNB)
- **Format:** Tabular network flow records (`MachineLearningCSV` collection), 78 statistical descriptors per flow (packet length variance, port profiles, packet counts, directional flags, etc.)
- **Files used:** `Tuesday-WorkingHours.pcap_ISCX.csv` (BENIGN traffic) and `Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv` (BENIGN + DDoS traffic)

### Open-Set Split Strategy

| Class | Role |
|---|---|
| BENIGN | Known |
| FTP-Patator, SSH-Patator | Known |
| **DDoS** | **Hidden — held out entirely from training, used only to test unknown-class detection** |

The DDoS attack family is completely excluded from training and validation, and reserved strictly for the open-set testing phase, so it acts as a stand-in for an unseen, zero-day threat.

## Methodology

1. **Data cleaning** — concatenate source CSVs, strip whitespace from column names, drop `inf`/`NaN` rows.
2. **Closed-set labeling** — of the non-DDoS traffic, map `BENIGN → 0` and any other known attack → `1`.
3. **Preprocessing** — `StandardScaler` feature scaling, stratified 80/20 train/validation split.
4. **Baseline classifiers** — a `RandomForestClassifier` (n_estimators=50) as the primary model, compared against a `LogisticRegression` baseline.
5. **Open-set calibration** — the Random Forest's per-sample `predict_proba` confidence is used to calibrate a rejection threshold (5th percentile of validation confidence scores). Any prediction whose top confidence falls below this threshold is screened out as **"Unknown/Anomaly" (class 2)**, rather than forced into a known category.
6. **Evaluation** — confusion matrix, ROC/AUC, and classification report on (a) the known-class validation split and (b) a live demo stream mixing held-out BENIGN traffic with the fully unseen DDoS traffic.

## Results

| Model | Validation Accuracy (known classes) |
|---|---|
| Logistic Regression | 99.34% |
| Random Forest (baseline) | 99.996% |

**Open-set screening on the live demo stream** (1,000 BENIGN + 1,000 unseen DDoS flows):

| Class | Precision | Recall | F1 |
|---|---|---|---|
| Benign (0) | 0.71 | 1.00 | 0.83 |
| Unfamiliar Threat (2) | 0.99 | 0.59 | 0.74 |

Overall accuracy on the mixed known/unknown stream: **80%**. The model successfully screens the majority of never-before-seen DDoS traffic as anomalous purely from confidence calibration, with no DDoS examples used in training.

## Limitations

- Only DDoS is treated as an unseen attack; other attack families were not tested as the "hidden" class.
- Random Forest confidence scores are not perfectly calibrated probabilities, which limits precision of the rejection threshold.
- Evaluation used a memory-efficient subset of CICIDS2017 (2 of 8 available days), not the full dataset.
- Future work should evaluate multiple hidden attack families and better-calibrated open-set methods (e.g., OpenMax, temperature scaling).

## Repository Structure

```
.
├── notebook.ipynb       # Full Colab notebook: EDA, training, open-set calibration, evaluation
├── requirements.txt     # Python dependencies
├── README.md
└── LICENSE
```

## Running the Project

This notebook was built for Google Colab and expects the CICIDS2017 CSVs in Google Drive.

1. Open `notebook.ipynb` in [Google Colab](https://colab.research.google.com/).
2. Upload the CICIDS2017 `MachineLearningCSV` files to your Google Drive (default expected path: `MyDrive/Machine_learning`), or edit `DRIVE_DATA_DIR` in the first code cell to match your folder.
3. Run all cells in order. The trained model and scaler are cached locally as `open_set_rf_model.pkl` and `scaler_transform.pkl` for the live demo section.

To run locally instead of Colab, install dependencies from `requirements.txt`, remove the `google.colab.drive` mount cell, and point `DRIVE_DATA_DIR` at a local folder containing the CSVs.

## Conclusion

A Random Forest–based Open-Set Learning framework was developed for network intrusion detection. DDoS attacks were completely excluded from training and treated as previously unseen threats. Using confidence-based rejection, the model successfully screened suspicious network traffic that did not resemble known patterns, demonstrating the potential of Open-Set Learning for catching emerging cyberattacks unavailable during training.
