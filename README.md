# **DCPC Friction & Spending — Causal Analysis (Double ML + DR-Loss)**

This repository implements a complete causal pipeline for studying **how low-friction payment methods (credit/debit/ACH/OBBP)** affect **daily spending** using the **2024 Atlanta Fed Diary of Consumer Payment Choice (DCPC)** data.

Our focus is on recovering the **causal effect** of “friction” on spending using **Double Machine Learning (DML)**, **orthogonal DR-loss**, and **CATE estimation**, while adapting the pipeline to the strong constraints of the dataset (only 4 diary days per person, non-random diary assignment, sparse demographic variation).

---

## **Repository Structure**

```
dcpc-friction-dml/
│
├── data/
│   ├── dcpc_2024_tranlevel_public.csv
│   ├── dcpc_2024_daylevel_public.csv
│   ├── dcpc_2024_indlevel_public.csv
│
├── notebooks/
│   ├── 01_data_overview.ipynb
│   ├── 02_build_inperson_day.ipynb
│   ├── 03_dml_inperson_main.ipynb
│   ├── 04_dr_loss_rich.ipynb
│
├── outputs/
│   └── day_inperson_ready.csv (02 results)
│
├── figures/
│   └── (Will for the figures from 03 and 04)
│
└── README.md
```

---

## **Notebook Guide**

### **01_data_overview.ipynb**

* Loads all three raw DCPC files (tran/day/ind).
* Performs schema checks, null-rate inspection, and ID matching.
* Confirms each participant has exactly **4 diary days**.
* Sanity-checks transaction amounts and diary consistency.

---

### **02_build_inperson_day.ipynb**

* Filters the dataset to **in-person payments only** (to avoid online/mobile ambiguity).

* Applies the **strict friction classification**:

  * **Low-friction**: {3,4,6,7,…} → credit/debit/ACH/OBBP, etc.
  * **High-friction**: {1,2,8} → cash/check/money order
  * **Discarded**: multi-method, PayPal residues, transfers, missing PI, etc.

* Aggregates to a day-level panel:

  * total amount
  * number of transactions
  * low-friction share (`T_share_low`)
  * weekday fixed effects
  * merchant-category shares
  * daily active-person weights (`w_day`)
  * price cap filters (≤150 / ≤200)

* Exports **day_inperson_ready.csv**.

---

### **03_dml_inperson_main.ipynb**

Main causal estimation using **Double Machine Learning**:

* Treatment:
  **`T_share_low_trim` = low-friction share clipped to [0.05, 0.95]**

* Outcome:
  **`Y_log_amt` = log(1 + daily total spending)**

* Controls:
  weekday, merchant-share vectors, and optional demographic merge.

* Performs:

  * Cross-fitted DML (Random Forest residualization)
  * Cluster bootstrap at the person level
  * Mechanism analysis (avg-ticket effect)
  * Robustness with small-ticket days only

**Core result:**
Low-friction share ↑ 10pp → spending ↑ **~5%** (stable across specs).

---

### **04_dr_loss_rich.ipynb**

Orthogonal **DR-loss** implementation + **CATE estimation**:

* Learns both:

  * global causal effect θ
  * heterogeneous effects τ(X)

* Produces:

  * Global DR-loss estimate (≈ DML result)
  * Distribution of τ̂
  * τ̂ vs weekday
  * τ̂ vs avg-ticket tercile
  * DR-loss global objective value

**Findings:**

Higher-spending and weekend contexts show **larger marginal effects** of low-friction payment use.

---

## **Key Features**

* Full pipeline in Jupyter notebooks (easy to extend or modify).
* Strict friction coding to avoid payment-type ambiguity.
* Person-cluster bootstrapping for valid inference.
* Orthogonal ML (DML + DR-loss) to handle high-dimensional controls.
* CATE analysis to reveal heterogeneity in behavioral responses.

---

## **Results (Short Summary)**

Across all specifications:

* **Low-friction payment use causally increases spending.**
* Point estimate is extremely stable:

  * **≈ 5% more spending for every +10pp increase** in low-friction share.
* Stronger effects:

  * for higher-ticket days,
  * for weekend transactions.

---

## **Reproducibility**

To re-run the full pipeline:

1. Place all three DCPC CSV files under `data/`
2. Open notebooks in order:
   **01 → 02 → 03 → 04**
3. Outputs (cleaned data + figures) will appear in the repo.

