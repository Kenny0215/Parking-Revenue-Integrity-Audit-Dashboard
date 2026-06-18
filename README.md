# NaN Sense: Parking Revenue Integrity Audit & Predictive Dashboard

An end-to-end data science and business intelligence solution designed to audit parking transaction integrity, predict revenue leakage, and optimize municipal enforcement resources for local councils (MBMB & MPHTJ).

---

## 📌 Project Overview

Local municipal councils face significant financial losses due to **parking revenue leakage**—instances where drivers occupy public parking bays but fail to pay, or overstay past their active tickets. 

**NaN Sense** is a complete data science pipeline and analytical solution that:
1. **Audits Historical Integrity:** Cross-references hourly parking payment logs against physically issued code violations/compounds to identify revenue gaps.
2. **Predicts Real-time Revenue Risk:** Trains a robust Machine Learning model (**Random Forest**) to predict the probability of revenue leakage (non-compliance) based on temporal, geospatial, and behavioral patterns.
3. **Deploys Actionable Tools:** Provides a high-fidelity **Power BI Desktop Dashboard** for strategic planning, and a **Gradio Web App** for operational "What-If" simulation by parking enforcement managers.

---

## 🛠️ Tech Stack & Architecture

* **Data Preprocessing & ML Pipeline:** Python, Pandas, Numpy, Scikit-Learn, XGBoost, Joblib (Google Colab)
* **Business Intelligence & Visualization:** Power BI Desktop, DAX (Data Analysis Expressions)

---

## 📊 The Data Pipeline & Preprocessing

The raw data consists of two distinct operational files:
* `MBMB_PARKING_FINAL.csv`: Active payment transaction logs.
* `MBMB_COMPOUND.csv`: Physically issued code violations (non-payment/overstay tickets).

### The Left-Join Strategy
Previously, an *Inner Join* was used to match tables, which accidentally threw away drivers who never opened the parking app at all (100% Unpaid Leakage). 

We engineered an inclusive **Left-Join Pipeline** (with Compounds as the base table) to ensure both categories of leakage are preserved and modeled:
1. **Overstay Leakage (Case A):** Driver paid, but the compound timestamp is after the parking expiry time.
2. **Completely Unpaid Leakage (Case B):** Driver parked and was fined, but has no payment transaction record (features default safely to `0` or `-1`).

### Engineered Features (`X`)
* `parking_hour`: Hour of the day the session started (0–23).
* `day_of_week`: Day of the week (0 = Monday, 6 = Sunday).
* `is_weekend`: Binary flag (1 = Weekend, 0 = Weekday).
* `month`: Calendar month of the year (1–12).
* `paid_duration_min`: Actual purchased parking duration in minutes.
* `price_rm`: Amount paid by the driver (RM).
* `is_unpaid`: Binary flag indicating zero payment.
* `council_enc` / `zone_enc` / `road_name_enc`: Label-encoded categorical location data.

---

## 🤖 Machine Learning Model Performance

We trained three distinct classifiers—**Random Forest**, **XGBoost**, and **Logistic Regression**—on chronologically ordered data (`shuffle=False` to prevent data leakage in time-series evaluation). 

**Random Forest** emerged as the champion model due to its robust handling of minority-class classification and class weighting:

* **AUC-ROC Score:** `0.803` (Excellent class separation capability).
* **F1-Score / Recall Balance:** Outstanding performance on highly imbalanced real-world violations.

### Feature Importance (The "Why")
Our feature importance analysis proved that **Street Name (`road_name_enc`)** and **Time of Day (`parking_hour`)** are the primary drivers of parking non-compliance. This indicates that leakage is heavily tied to specific urban hotspots and peak-hour congestion.

---

## 📈 Power BI Desktop Dashboard

The dashboard is designed as a unified, scrollable single-page report consisting of four functional sections:

### 1. Executive KPIs
* **Total Sessions:** The total volume of audited parking events (1,871 cases).
* **Match Rate:** The percentage of fully compliant drivers (85.14%).
* **Leakage Sessions:** Total historical cases where non-compliance occurred (278 sessions).
* **Predicted Loss:** Financial risk calculated by the AI model based on current patterns.

### 2. Spatiotemporal Analysis
* **Leakage Heatmap (Day x Hour):** A color-scaled matrix proving that **Friday mornings** are the highest-risk times for revenue leakage.
* **Leakage by Zone/Area:** Bar chart identifying **Bandar Hilir** and **Kota Laksamana** as the top geographical hotspots requiring patrol deployment.

### 3. Predictive Comparison Timeline
* **Actual vs. Predicted Revenue Loss (Line Chart):** Compares historical financial loss against the AI's predictions month-by-month. The tight tracking of both lines validates the model's high real-world accuracy.

### 4. AI Model & Operational Audit
* **Council & Product Type Comparison:** Provides targeted metrics comparing MBMB vs. MPHTJ leakage rates, and ranks which ticket products are most abused.

---
