# Mini Group Project I – Investigating Agricultural Burning with Remote Sensing Data

### **Course:** CNGF5020  
### **Region of Study:** Heilongjiang Province, China  
### **Contributors:** Group Members (replace with names)  
### **Due Date:** November 5, 2025  

---

## Overview  

This project examines **agricultural residue burning** in Heilongjiang Province using multi-source remote sensing data.  
The study integrates **MODIS active fire detections**, **FY straw-burning monitoring data**, and **crop phenology rasters** to identify, classify, and analyze burning events at spatiotemporal and policy levels.  

The analysis is organized into four main tasks:  
- **Task 1:** Spatiotemporal patterns of fire activity (2010–2019)  
- **Task 2:** Data cleaning and temporal link with crop maturity  
- **Task 3:** Long-term trend and turning point analysis  
- **Task 4:** Machine learning classification of agricultural fires  

---

## Repository Structure  

```
├── task1.ipynb              # Spatiotemporal pattern analysis (2010–2019)
├── task2.ipynb        # Data preprocessing and crop–fire temporal linkage
├── task3.ipynb              # Long-term trend and turning point analysis
├── task4.ipynb        # Gradient Boosting fire classification
├── data/
│   ├── modis_2010_China.csv
│   ├── Heilongjiang_Maize_MA_2010.tif
│   ├── Heilongjiang_Wheat_MA_2010.tif
│   ├── straw burning fire point monitoring data.xlsx
│   └── CHN_County.shp
└── README.md                # Final project report
```

---

## Objectives  

1. Explore spatiotemporal patterns of fire activity across Heilongjiang (2010–2019).  
2. Link fire occurrences to crop maturity and identify likely agricultural fires.  
3. Quantify long-term changes and turning points in burning intensity and frequency.  
4. Build a machine-learning classifier for automatic fire-type detection.  

---

## 🔹 Task 1 – `task1.ipynb` Spatiotemporal Distribution of Fire Activity (2010–2019)

### **Objective**  
Identify dominant temporal and spatial patterns of fire occurrence in Heilongjiang.

### **Methods**  
1. **Data & Pre-processing**  
   - MODIS Active Fire data (`modis_20XX_China.csv`, 1 km, 4× per day); fields: latitude, longitude, acq_date, confidence.  
   - Spatial filter: 121–135°E, 43.5–53.5°N (Heilongjiang extent).  
   - Time fields: derive year, DOY, and week.  
2. **Analysis with pandas + numpy**  
   - Weekly curve: aggregate fires by week/DOY to locate seasonal “hot weeks.”  
   - Spatial density: `numpy.histogram2d` + `pcolormesh` to map 1 km grid frequency.  
   - County statistics: manual bounding-boxes for 13 prefectures (no GeoPandas required).  
3. **Key Findings**  
   - Peak activity occurs in Weeks 39–41 (late September – early October), matching maize and wheat harvest.  
   - Hotspots concentrate along the Harbin–Daqing–Qiqihar industrial belt and Sanjiang Plain, indicating post-harvest burning.  

---

## 🔹 Task 2 – `task2.ipynb` Data Cleaning & Temporal Analysis  

*(content unchanged from previous version)*  

---

## 🔹 Task 3 – `task3.ipynb` Long-Term Trend and Turning Point Analysis (2010–2019)

### **Objective**  
Quantify county-level changes in burning rate and intensity over a decade, and detect policy-related turning points.

### **Methods**  
1. **Supplementary Data**  
   - Crop maturity rasters: `Heilongjiang_Maize_MA_2010.tif` & `Wheat_2010.tif` (1 km DOY).  
   - County frames: 13 prefecture bounding boxes defined manually (lat/lon).  
   - Fire Radiative Power (FRP) used as intensity indicator when available.  
2. **Analytical Workflow (without GeoPandas / rasterio)**  
   - Direct NumPy reading of TIFF arrays with hard-coded GeoTransform.  
   - Crop pixels by county bounding boxes.  
   - **Burning rate:**  
     `burn_rate = (annual fires / crop pixels) × 1000` (fires per thousand pixels, area-normalized).  
   - **Trend testing:** Theil–Sen slope + Mann–Kendall p-value (α = 0.05).  
   - **Turning point:** detect maximum residual drop year from provincial time series.  
   - **Visualization:**  
     county bars (blue = improvement, red = worsening, gray = insignificant); time series with red dashed break year.  
3. **Main Findings**  
   - ≈70 % of counties show no significant trend (p ≥ 0.05); declines mainly in the Harbin–Daqing–Qiqihar belt.  
   - 2017 identified as province-wide turning point; post-2017 fires fell > 60 %, aligning with policy enforcement.  
   - Average FRP also decreased ≈22 %, indicating fewer and smaller burning events.  

---

## 🔹 Task 4 – `task4.ipynb` Machine Learning Classification  

*(content unchanged from previous version)*  

---

## Summary of Insights  

- Fire activity peaks coincide with harvest periods (Weeks 39–41).  
- Post-2017 policy enforcement significantly reduced both fire frequency and intensity.  
- Machine-learning classification achieved high accuracy in distinguishing agricultural burning events.  

---

## Environment & Dependencies  

```bash
Python >= 3.9 (tested on 3.10.12)  
pandas  
numpy  
geopandas  
rasterio  
matplotlib  
seaborn  
scikit-learn  
datetime
```

---

## Limitations & Future Work  

- Crop maturity datasets use fixed DOY values, introducing uncertainty.  
- MODIS 4× daily observations may miss short-duration fires.  
- Future work should integrate meteorological (wind, humidity) and pollution (PM2.5) data to assess environmental impacts.  
