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

* **Sumber Data:** `dynamic_supply_chain_logistics_dataset.csv`
* **Variabel Target (Y):** **`is_delayed`**
    * Target ini didefinisikan secara biner: **1** jika variasi waktu kedatangan yang diperkirakan (`eta_variation_hours`) **lebih dari 3 jam**, dan **0** jika tidak (Tepat Waktu).

---

## Metodologi dan Pipeline

Proyek ini mengadopsi *pipeline* yang ketat, dirancang untuk mengatasi *class imbalance* dan kompleksitas data logistik.

### 1. Preprocessing & Feature Engineering

1.  **Capping Outliers:** Menggunakan metode **1.5 * IQR** untuk membatasi nilai ekstrem (outliers) pada fitur-fitur numerik.
2.  **Feature Engineering Lanjutan:**
    * **Fitur Pergerakan:** Perhitungan jarak tempuh (`distance_km`) dan kecepatan rata-rata (`speed_kmph`) menggunakan **Formula Haversine** dan pembuatan fitur tren bergulir (`speed_kmph_rolling3`).
    * **Fitur Risiko Agregat:** Pembuatan metrik interaksi risiko seperti `traffic_weather_interaction`, `port_traffic_combo`, dan `operational_risk_combo`.
3.  **Data Split:** Digunakan **Time-Series Split (80% Train, 20% Test)**, di mana data latih adalah data lama dan data uji adalah data terbaru.

### 2. Model & Resampling

| Komponen Pipeline | Teknik yang Dipilih | Fungsi |
| :--- | :--- | :--- |
| **Preprocessing** | `ColumnTransformer` (Standardisasi + OHE) | Menjaga konsistensi transformasi data latih dan uji. |
| **Resampling** | **SMOTEENN** | Teknik *hybrid* (SMOTE + ENN) untuk menyeimbangkan distribusi kelas pada data latih. |
| **Model Utama** | **Balanced Random Forest Classifier (BRFC)** | Model *ensemble* kuat yang dilatih untuk memprediksi probabilitas keterlambatan. |

---

## Early Warning System (EWS) Output

Hasil prediksi probabilitas model dikonversi menjadi laporan EWS yang dapat ditindaklanjuti.

### 1. Risk Level Mapping

Probabilitas keterlambatan (`Delay_Probability`) dikategorikan sebagai berikut:

| Probabilitas Keterlambatan | Tingkat Risiko |
| :--- | :--- |
| **$ < 0.33 $** | **Low Risk** |
| **$ 0.33 \le \text{Prob} < 0.66 $** | **Moderate Risk** |
| **$ \ge 0.66 $** | **High Risk** |

### 2. Output Laporan

* **File Output:** **`shipment_early_warning_report.csv`**
* Laporan diurutkan secara **menurun** berdasarkan `Delay_Probability`, sehingga kasus **risiko tertinggi** (potensi intervensi) selalu berada di urutan teratas.
* **Kolom Laporan Kunci:** `timestamp`, `vehicle_gps_latitude`, `route_risk_level`, `Risk_Level`, `Delay_Probability`, dan `Actual_Delay_Status`.

---

## Interpretasi Model (SHAP)

SHAP digunakan untuk menjelaskan kontribusi setiap fitur terhadap prediksi, memastikan transparansi model.

### 1. SHAP TreeExplainer

* `shap.TreeExplainer` diinisialisasi pada model BRFC yang telah dilatih.
* Nilai SHAP dihitung pada data uji yang sudah diproses, fokus pada **Kelas 1 (Terlambat)**.

### 2. Interpretasi Visual

| Plot SHAP | Tujuan | Insight yang Diberikan |
| :--- | :--- | :--- |
| **Bar Plot** | Menunjukkan **pentingnya fitur global** secara keseluruhan. | Fitur mana yang memiliki dampak rata-rata terbesar pada prediksi keterlambatan. |
| **Beeswarm Plot** | Menunjukkan **dampak individual dan arah** nilai fitur. | Nilai fitur yang tinggi (merah) atau rendah (biru) mendorong prediksi ke arah **Terlambat (kanan)** atau **Tepat Waktu (kiri)**. |



---

## Dependencies (Library yang Dibutuhkan)

* `pandas`
* `numpy`
* `matplotlib`
* `seaborn`
* `sklearn` (Preprocessing, Metrics, Model Selection)
* `imblearn` (`SMOTEENN`, `ImbPipeline`)
* `shap` (Interpretasi Model)

---

## Cara Menjalankan

1.  Pastikan semua *dependencies* di atas telah terinstal.
2.  Letakkan file `dynamic_supply_chain_logistics_dataset.csv` dalam direktori proyek.
3.  Jalankan *notebook* **`Kelompok_12_FP_datmin.ipynb`** secara berurutan.
4.  Laporan akhir akan disimpan sebagai **`shipment_early_warning_report.csv`**.
