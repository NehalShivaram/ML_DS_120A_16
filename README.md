# ML Project 6: Unknown-Attack Screening in Network Flow Data Using Open-Set Learning

Applied Machine Learning (Summer 2026) · Team ID: ML_DS_120A_16
Team: Dhruv Sanjay Patel, Nehal Shivaram

## Project Overview

This repository contains the implementation and results for **Unknown-Attack Screening in Network Flow Data Using Open-Set Learning**.

Traditional network intrusion detection systems operate under a closed-set assumption, struggling to flag novel or modified exploits. This project builds a robust Open-Set Learning framework capable of classifying known network behaviors while isolating unfamiliar, zero-day threat families — without any prior exposure to the hidden attack during training.

The notebook includes data loading and cleaning, EDA, an anti-leakage open-set train/test split, baseline model training (Random Forest and Logistic Regression), confidence-based open-set threshold calibration, evaluation (confusion matrix, ROC/AUC, classification reports), and a live demo pipeline.

- **Dataset:** CICIDS2017 (671,356 flow records across BENIGN, DDoS, FTP-Patator, SSH-Patator)
- **Hidden ("zero-day") attack family:** DDoS — entirely excluded from training
- **Objective:** Train on known traffic patterns, then correctly flag DDoS traffic as unfamiliar at test time using Open-Set Learning

## Repository Structure

```
.
├── notebook.ipynb
├── Figures/
│   ├── figure_1_class_distribution.png
│   ├── figure_2_feature_correlation_heatmap.png
│   ├── figure_3_top20_feature_importance.png
│   ├── figure_4_openset_confusion_matrix.png
│   ├── figure_5_openset_roc_curve.png
│   └── figure_6_threshold_sensitivity.png
├── Tables/
│   ├── table_1_model_comparison.csv
│   ├── table_2_top20_feature_importance.csv
│   ├── table_3_openset_validation_report.csv
│   └── table_4_live_demo_report.csv
├── Models/
│   ├── open_set_rf_model.pkl
│   └── scaler_transform.pkl
├── Datasets/
│   └── README.md
├── requirements.txt
├── LICENSE
└── .gitignore
```

## Methodology

1. **Data cleaning** — concatenate source CSVs, strip whitespace from column names, drop `inf`/`NaN` rows. Result: 671,356 flow records, 78 features.
2. **Open-set split** — DDoS is held out entirely; remaining classes are mapped `BENIGN → 0`, other known attacks (FTP-Patator, SSH-Patator) → `1`. `StandardScaler` + stratified 80/20 train/validation split (434,664 train / 108,667 validation rows).
3. **Baseline models** — `RandomForestClassifier` (n_estimators=50) as the primary model, compared against `LogisticRegression`.
4. **Open-set calibration** — the Random Forest's `predict_proba` confidence calibrates a rejection threshold (5th percentile of validation confidence). Predictions below the threshold are screened as **Unknown/Anomaly (class 2)**.
5. **Evaluation** — confusion matrix, ROC/AUC, classification report on the known-class validation split and on a live demo stream mixing held-out BENIGN traffic with the fully unseen DDoS traffic.

## Results

*(Regenerated end-to-end from the actual CICIDS2017 CSVs.)*

| Model | Validation Accuracy (known classes) |
|---|---|
| Logistic Regression | 99.34% |
| Random Forest (baseline) | 99.995% |

**Open-set screening on the validation split** (calibrated threshold ≈ 1.0000, 5th percentile of confidence):

| Class | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Benign (0) | 1.00 | 0.97 | 0.99 | 105,901 |
| Known Attack (1) | 1.00 | 0.68 | 0.81 | 2,766 |
| Screened Anomaly (2) | 0.00 | 0.00 | 0.00 | 0 |

Rejected samples: 3,659 (rejection rate 3.37%).

**Open-set screening on the live demo stream** (1,000 BENIGN + 1,000 unseen DDoS flows):

| Class | Precision | Recall | F1 |
|---|---|---|---|
| Benign (0) | 0.73 | 0.96 | 0.83 |
| Unfamiliar Threat (2) | 0.95 | 0.65 | 0.77 |

Overall accuracy on the mixed known/unknown stream: **80.6%**. The model flags the majority of never-before-seen DDoS traffic as anomalous purely from confidence calibration, with zero DDoS examples used in training. Full tables are in `Tables/`, and plots (class distribution, correlation heatmap, feature importance, confusion matrix, ROC curve, threshold sensitivity) are in `Figures/`.

**Top predictive features:** Destination Port, Fwd Packet Length Max, Init_Win_bytes_backward, Average Packet Size, Packet Length Std — see `Tables/table_2_top20_feature_importance.csv` for the full ranking.

## Limitations

- Only DDoS is treated as an unseen attack.
- Random Forest confidence values are not perfectly calibrated.
- The system was evaluated on a subset of CICIDS2017 (2 of 5 days).
- Future work should evaluate multiple hidden attack families.

## Running the Project

This notebook was built for Google Colab and expects the CICIDS2017 CSVs in Google Drive.

1. Open `notebook.ipynb` in [Google Colab](https://colab.research.google.com/).
2. Get the dataset per `Datasets/README.md` and place it where `DRIVE_DATA_DIR` in the first code cell expects it (default: `MyDrive/Machine_learning`), or edit that path.
3. Run all cells in order. Trained model artifacts are cached to `Models/` as `open_set_rf_model.pkl` and `scaler_transform.pkl` for the live demo section — these are already included in this repo from the reference run.

To run locally instead of Colab: install dependencies from `requirements.txt`, remove the `google.colab.drive` mount cell, and point `DRIVE_DATA_DIR` at a local folder.

## Conclusion

A Random Forest–based Open-Set Learning framework was developed for network intrusion detection. DDoS attacks were completely excluded from training and treated as previously unseen threats. Using confidence-based rejection, the model successfully screened suspicious network traffic that did not resemble known patterns, demonstrating the potential of Open-Set Learning for catching emerging cyberattacks unavailable during training.
