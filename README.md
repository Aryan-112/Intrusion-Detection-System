# 🛡️ Network Intrusion Detection System (NIDS)

A machine learning pipeline for detecting network attacks using the **UNSW-NB15** cybersecurity benchmark dataset. The project compares four classification models and applies **SHAP explainability** to identify which network traffic features drive attack predictions.

---

## 📌 Overview

Traditional rule-based intrusion detection systems struggle to keep pace with evolving attack patterns. This project trains and evaluates ML models to classify network traffic as **Normal** or **Attack** across 9 distinct attack categories, using a real-world benchmark dataset from the Australian Centre for Cyber Security (ACCS).

The pipeline addresses key real-world ML challenges including class imbalance, categorical encoding of network protocols, and model interpretability — going beyond standard classification to surface *why* a prediction was made.

---

## 📂 Dataset

- **Source:** [UNSW-NB15](https://research.unsw.edu.au/projects/unsw-nb15-dataset) — created by the Australian Centre for Cyber Security, widely used in academic network security research
- **Training set:** 82,332 records
- **Testing set:** 175,341 records
- **Features:** 43 network traffic attributes including packet bytes, TTL values, connection state, protocol type, jitter, load, TCP window sizes, and HTTP transaction metadata
- **Target:** Binary — `0 = Normal`, `1 = Attack`

**Attack categories in the dataset:**
`Generic` · `Exploits` · `Fuzzers` · `DoS` · `Reconnaissance` · `Analysis` · `Backdoor` · `Shellcode` · `Worms`

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| Python | Core language |
| Pandas / NumPy | Data manipulation |
| Scikit-learn | Preprocessing, modelling, evaluation |
| XGBoost | Primary classification model |
| imbalanced-learn | SMOTE for class imbalance |
| SHAP | Model explainability |
| Matplotlib | Visualisation |
| Joblib | Model serialisation |

---

## ⚙️ Pipeline

### 1. Data Loading
- Official UNSW-NB15 training and testing splits loaded separately
- Identifier and leakage columns dropped: `id`, `srcip`, `dstip`, `stime`, `ltime`
- Multi-class attack category (`attack_cat`) retained for SMOTE; binary label (`label`) used for final evaluation

### 2. Preprocessing
- Missing values filled with `'None'` for features, `'Normal'` for attack category
- **One-hot encoding** applied to categorical columns: `proto`, `state`, `service`
- Train/test column alignment via `.align()` to handle categories present in one split but not the other
- **StandardScaler** fit on training data only, then applied to both sets

### 3. Class Imbalance — SMOTE
- Class imbalance is significant: rare categories like Worms (44 samples) and Shellcode (378) are heavily outnumbered by Generic (18,871) and Normal (37,000)
- **SMOTE** applied at the multi-class level before converting to binary labels — this ensures synthetic samples are generated for each attack type individually, rather than treating all attacks as one group
- Balanced training data converted to binary: `Normal = 0`, all attack types = `1`

### 4. Model Training & Comparison

Four models trained on SMOTE-balanced data and evaluated on the original test set:

| Model | Key Parameters |
|-------|---------------|
| XGBoost | `max_depth=8`, `learning_rate=0.1`, `n_estimators=300`, `subsample=0.8`, `colsample_bytree=0.8` |
| Random Forest | `n_estimators=100`, `max_depth=5`, `min_samples_split=10`, `class_weight='balanced'` |
| Decision Tree | `max_depth=12`, `class_weight='balanced'` |
| Logistic Regression | `max_iter=1000`, `class_weight='balanced'` |

All four models evaluated on: **Accuracy, Precision, Recall, F1-Score** with results visualised in a grouped bar chart.

### 5. Explainability — SHAP
- **SHAP TreeExplainer** applied to the XGBoost model on a 1,000-record sample of the test set
- **Global summary plot** — shows which features most influence attack classification across all predictions, with directionality (high/low feature values pushing toward attack or normal)
- **Local waterfall plot** — explains a single prediction, showing each feature's contribution to pushing the model above or below the decision threshold
- Top 3 SHAP features extracted per alert as a ranked contribution table

### 6. Model Export
Artefacts saved for downstream deployment:
```
xgboost_quantum_sec.pkl    # Trained XGBoost model
scaler_quantum_sec.pkl     # Fitted StandardScaler
features_quantum_sec.pkl   # Feature name list for inference alignment
```

---

## 📊 Results

XGBoost selected as the best-performing model based on overall metrics across accuracy, precision, recall, and F1-score. Full classification reports and confusion matrix generated as notebook outputs.

---

## 📁 Repository Structure

```
├── Main.ipynb                      # Full pipeline notebook
├── UNSW_NB15_training-set.csv      # Training data (82,332 records)
├── UNSW_NB15_testing-set.csv       # Testing data (175,341 records)
├── xgboost_quantum_sec.pkl         # Exported model (generated on run)
├── scaler_quantum_sec.pkl          # Exported scaler (generated on run)
├── features_quantum_sec.pkl        # Exported feature list (generated on run)
└── README.md
```

---

## 🚀 Future Work

- Build a Streamlit dashboard surfacing SHAP explanations as human-readable alert descriptions for security analysts
- Extend to multi-class classification to identify specific attack type rather than binary detection
- Evaluate generalisation on additional NIDS benchmarks (e.g. CIC-IDS2018)
- Explore anomaly detection approaches for zero-day attack identification

---

## 👤 Author

**Aryan Raheja**  
Bachelor of Computing Science (AI & Data Analytics), University of Technology Sydney
