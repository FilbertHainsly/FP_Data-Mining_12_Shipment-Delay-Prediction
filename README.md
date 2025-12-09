# Machine Learning Based Shipment Delay Prediction and Early Warning System (EWS)

## Deskripsi Proyek

Proyek ini bertujuan untuk membangun sebuah **Sistem Peringatan Dini (Early Warning System - EWS)** berbasis *Machine Learning* yang mampu memprediksi probabilitas keterlambatan pengiriman dalam rantai pasok.

Dengan menganalisis data operasional yang dinamis dan kompleks, sistem ini mengklasifikasikan pengiriman ke dalam tingkat risiko **Low, Moderate, atau High Risk** dan menyediakan interpretasi model menggunakan **SHAP** untuk transparansi pengambilan keputusan.

Tujuan akhirnya adalah memfasilitasi intervensi proaktif oleh tim operasional untuk mengurangi keterlambatan dan biaya terkait.

## Anggota Kelompok

| Nama | NRP | Kelompok |
| :--- | :--- | :--- |
| Badruzzaman Nafiz | 5025231196 | 12 |
| Daffa Rinali | 5025231209 | 12 |
| Filbert Hainsly Martin | 5025231256 | 12 |

---

## Dataset & Target
* **Link Dataset :** [https://www.kaggle.com/datasets/datasetengineer/logistics-and-supply-chain-dataset](https://www.kaggle.com/datasets/datasetengineer/logistics-and-supply-chain-dataset)
* **Sumber Data:** `dynamic_supply_chain_logistics_dataset.csv`
* **Variabel Target (Y):** **`is_delayed`**
    * Target ini didefinisikan secara biner: **1** jika variasi waktu kedatangan yang diperkirakan (`eta_variation_hours`) **lebih dari 3 jam** (Terlambat) dan **0** (Tepat Waktu).

---

## Metodologi dan Pipeline

Proyek ini mengadopsi *pipeline* yang ketat, dirancang untuk mengatasi *class imbalance* dan kompleksitas data logistik.

### Preprocessing & Feature Engineering

1.  **Capping Outliers:** Menggunakan metode **1.5 * IQR** untuk membatasi nilai ekstrem (outliers) pada fitur-fitur numerik.
2.  **Feature Engineering Lanjutan:**
    * **Fitur Pergerakan:** Perhitungan jarak tempuh (`distance_km`) dan kecepatan rata-rata (`speed_kmph`) menggunakan **Formula Haversine** dan pembuatan fitur tren bergulir (`speed_kmph_rolling3`).
    * **Fitur Risiko Agregat:** Pembuatan metrik interaksi risiko seperti `traffic_weather_interaction`, `port_traffic_combo`, dan `operational_risk_combo`.
3.  **Data Split:** Digunakan **Time-Series Split (80% Train, 20% Test)**, di mana data latih adalah data lama dan data uji adalah data terbaru.


## Model yang Digunakan

Proyek ini menguji performa 8 model klasifikasi yang dikombinasikan dengan berbagai skenario penanganan ketidakseimbangan kelas (*imbalance*).

### 1. Daftar Model Klasifikasi (8 Model)

1.  **Logistic Regression (LR):** Model linier sebagai *baseline*.
2.  **K-Nearest Neighbors (KNN):** Model berbasis jarak.
3.  **Decision Tree (DT):** Model berbasis aturan.
4.  **Random Forest (RF):** Model *ensemble* berbasis *bagging*.
5.  **Gradient Boosting (GB):** Model *ensemble* berbasis *boosting*.
6.  **XGBoost (XGB):** Model *boosting* yang terdistribusi dan efisien.
7.  **Support Vector Machine (SVM):** Model berbasis *hyperplane* (dengan kernel).
8.  **Balanced Random Forest (BRF):** Model *ensemble* yang secara intrinsik menangani data tidak seimbang.

### 2. Teknik Resampling (7 Skenario)

Model-model di atas diuji di bawah 7 skenario *resampling* yang berbeda, di mana **Skenario 5 (SMOTEENN)** terbukti paling optimal.

| Skenario | Teknik | Kategori |
| :--- | :--- | :--- |
| **Skenario 1** | **No Balancing** | *Baseline* (Tanpa penanganan *imbalance*) |
| **Skenario 2** | Oversampling: **SMOTE** | Peningkatan sampel kelas minoritas. |
| **Skenario 3** | Undersampling: **RandomUnderSampler** | Pengurangan sampel kelas mayoritas. |
| **Skenario 4** | Hybrid: **SMOTE-Tomek** | Gabungan SMOTE dan *Tomek Links* (membersihkan *border*). |
| **Skenario 5** | **Hybrid: SMOTEENN (Terbaik)** | Gabungan SMOTE dan *Edited Nearest Neighbors* (membersihkan *noise*). |
| **Skenario 6** | Oversampling: **ADASYN** | Pembuatan sampel baru yang lebih fokus di area *border*. |
| **Skenario 7** | Ensemble: **EasyEnsemble** | Membuat beberapa *subset* seimbang dari data mayoritas. |

---

## Skenario Pengujian

### A. Metodologi Data Splitting

* **Pembagian Waktu (*Time-Series Split*):** Data dibagi menjadi **Data Latih (80% data lama)** dan **Data Uji (20% data terbaru)** berdasarkan urutan waktu. Metode ini penting untuk memastikan model dapat digeneralisasi pada data operasional di masa depan.

### B. Metrik Evaluasi

Karena data target sangat tidak seimbang, evaluasi difokuskan pada metrik yang sensitif terhadap kelas minoritas (Keterlambatan):

1.  **Recall (Sensitivitas):** Kemampuan model untuk mengidentifikasi semua kasus keterlambatan yang sebenarnya (minimasi *False Negatives*).
2.  **Precision:** Akurasi prediksi keterlambatan (minimasi *False Positives*).
3.  **F1-Score:** Rata-rata harmonik dari Precision dan Recall.
4.  **Area Under the ROC Curve (AUC-ROC):** Kemampuan diskriminasi model secara keseluruhan.

### C. Pemilihan Model Terbaik

Model terbaik dipilih dari kombinasi **8 Model x 7 Skenario** yang menghasilkan nilai **F1-Score tertinggi** pada *cross-validation* (atau data uji). **Kombinasi terbaik yang digunakan dalam EWS adalah Balanced Random Forest dengan SMOTEENN.**

---

## Early Warning System (EWS) Output

### 1. Pemetaan Risiko

Probabilitas keterlambatan (`Delay_Probability`) dikonversi menjadi kategori risiko dengan ambang batas yang telah ditentukan:

| Probabilitas Keterlambatan | Tingkat Risiko |
| :--- | :--- |
| **< 0.33** | **Low Risk** |
| **0.33 - 0.66** | **Moderate Risk** |
| **> 0.66** | **High Risk** |

### 2. Laporan Output

* **File Output:** **`shipment_early_warning_report.csv`**
* Laporan diurutkan secara **menurun** berdasarkan `Delay_Probability`, menempatkan pengiriman berisiko tertinggi di urutan teratas untuk tindakan mitigasi segera.

---

## Interpretasi Model (SHAP)

SHAP digunakan pada model terbaik (BRF + SMOTEENN) untuk memberikan penjelasan lokal dan global.

* **SHAP Bar Plot (Global Importance):** Menunjukkan 15 fitur utama yang paling penting secara global (rata-rata dampak absolut).
* **SHAP Beeswarm Plot (Impact and Direction):** Menjelaskan bagaimana nilai fitur yang tinggi atau rendah (merah/biru) secara spesifik mendorong prediksi kasus ke arah **Terlambat** atau **Tepat Waktu**.


---
