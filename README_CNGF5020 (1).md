# Mini Group Project I – Investigating Agricultural Burning with Remote Sensing Data

### Course  
**Group Members:** 

---

## 1. Project Overview

This project investigates **agricultural residue burning (straw burning)** in **Heilongjiang Province, China**, integrating multi-source remote sensing datasets including MODIS active fire points, FY (Fengyun) fire detection, and crop phenology rasters. Although straw burning is officially prohibited, identifying these events from natural wildfires remains a major challenge due to overlapping spatial and temporal signatures.

Our objective is to design a **data-driven workflow** to detect, classify, and interpret agricultural burning using open satellite data and machine learning. The study combines **spatiotemporal analysis**, **crop maturity alignment**, and **gradient boosting classification** to quantify agricultural burning and its trends between **2010–2019**.

---

## 2. Repository Structure

```
├── task1.ipynb        # Spatiotemporal analysis of MODIS fire points (2010–2019)
├── task2.ipynb        # Fire-crop maturity alignment and agricultural attribution
├── task3.ipynb        # Long-term trend and county-level change analysis
├── task4.ipynb        # Gradient Boosting classifier for fire type identification
├── data/
│   ├── modis_2010_China.csv
│   ├── Heilongjiang_Maize_MA_2010.tif
│   ├── Heilongjiang_Wheat_MA_2010.tif
│   ├── straw_burning_fire_point_monitoring.xlsx
│   ├── CHN_County.shp
└── README.md          # Final report (this document)
```

---

## 3. Datasets and Preprocessing

### 3.1 MODIS Active Fire Data
- **Spatial range:** 121–135°E, 43.5–53.5°N (Heilongjiang Province)  
- **Attributes used:** latitude, longitude, acq_date, frp, confidence  
- **Preprocessing:**  
  1. Filtered out missing values and low-confidence detections.  
  2. Converted time strings (e.g., `1230` → `12:30`) using `datetime`.  
  3. Derived temporal features: `year`, `DOY` (day of year), and `week`.  
  4. Removed outliers beyond province boundaries.  
  5. Combined date and time into ISO datetime for temporal aggregation.

### 3.2 FY Straw-Burning Monitoring Data
- **Source:** Validated by field surveys (August 2016 – February 2017).  
- **Cleaning:** Removed invalid or missing coordinates and clipped to Heilongjiang boundaries using `geopandas`.  
- Converted to `GeoDataFrame` for spatial overlay analysis.

### 3.3 Crop Phenology Raster (Heilongjiang_Maize_MA_2010.tif, Wheat_MA_2010.tif)
- Each pixel represents the **Day of Year (DOY)** when crops reach maturity.  
- Used to identify harvest windows for maize and wheat (DOY ≈ 270–290).  
- Combined with fire DOY to calculate the **time difference between fire and crop maturity (Δdays)**.

---

## 4. Analytical Workflow

### **Task 1 – Spatiotemporal Pattern Analysis (2010–2019)**
Objective: Identify seasonal and spatial hotspots of fire activity.  
Methods:
- Weekly aggregation of fire counts using `pandas` and `numpy`.  
- Heatmaps created via `numpy.histogram2d` + `pcolormesh`.  
- Visualized hotspot weeks and spatial clusters.  
Findings:
- Fire peaks occur between **weeks 39–41 (late Sep–early Oct)**, consistent with maize/wheat harvest.
- Hotspots concentrated in **Harbin–Daqing–Qiqihar corridor** and **Sanjiang Plain**, indicating post-harvest burning behavior.

---

### **Task 2 – Crop Maturity Alignment**
Objective: Quantify agricultural relevance by matching fires to crop maturity.  
Methods:
- Defined function `match_fire_to_crop()` taking three inputs: fire dataset, crop raster, and crop type.
- Computed `days_after_crop = fire_DOY - crop_DOY`.
- Focused on −30 ≤ Δdays ≤ +60 window to isolate potential straw burning events.
- Calculated percentage of fires within **1–30 days after maturity** (key straw-burning indicator).  
Findings:
- 62% of fires occurred within 30 days after crop maturity.  
- Mean Δdays ≈ +10 indicates most fires follow harvest.  
- Histogram visualization confirms a strong post-harvest clustering.

---

### **Task 3 – Long-Term Trend and Turning Point (2010–2019)**
Objective: Examine temporal changes in agricultural burning.  
Methods:
- Calculated annual **burning rate** = (fire count / crop area pixels) × 1000.  
- County-level trend testing via **Theil–Sen slope** and **Mann–Kendall test (α = 0.05)**.  
- Identified turning points from residual maxima of national fire sequences.  
Findings:
- ~70% of counties show statistically insignificant change;  
- Significant declines in **Harbin, Daqing, Qiqihar** after 2017;  
- **2017 marked a province-wide drop (>60%)** consistent with intensified enforcement of open burning bans;  
- Mean FRP decreased by 22%, suggesting reduced intensity.

---

### **Task 4 – Machine Learning Classification**
Objective: Automate classification of “agricultural” vs. “other” fires.  
Methods:
- Features: latitude, longitude, DOY, hour, FRP, crop type, and proximity metrics.  
- Labels: agricultural fires (within crop area + harvest window) vs others.  
- Model: **Gradient Boosting Classifier (GBM)**, 70/30 train-test split.  
- Standardized features for balanced learning.  
- Evaluated via accuracy, recall, and confusion matrix.  
Findings:
- **Accuracy: 85%**, Recall for agricultural fires: 0.82.  
- Feature importance: `DOY`, `FRP`, and `in_maize` dominated model decisions.  
- Classifier successfully learned human-like logic: “in crop field + near harvest = agricultural fire”.

---

## 5. Discussion and Challenges

### 5.1 FRP Intensity Analysis
Agricultural fires are generally **lower in intensity (mean FRP ≈ 20–40 MW)** compared to wildfires (>100 MW), implying shorter duration and smaller biomass load.

### 5.2 Methodological Uncertainty
- **Spatial mismatch:** 1 km MODIS resolution vs. fine-scale cropland mosaics.  
- **Temporal offset:** Fixed crop maturity maps (2010) may not reflect interannual variability.  
- **Validation limitation:** FY reference dataset covers only six months (2016–2017).  
Future improvement could integrate **Sentinel-2 NDVI phenology** or **LULC 30m maps** for higher precision.

### 5.3 Meteorological Considerations
Wind speed, humidity, and inversion layers influence fire detection probability and pollutant dispersion. Future work can correlate fire peaks with **PM2.5 or AOD datasets** to quantify atmospheric impacts.

---

## 6. Key Results Summary

| Aspect | Key Finding |
|--------|--------------|
| Fire Seasonality | Peak in weeks 39–41 (Sept–Oct) |
| Agricultural Attribution | 62% of fires within 30 days post-harvest |
| Long-term Trend | Significant decline after 2017 |
| Model Accuracy | 85% (GBM classification) |
| FRP Intensity | Agricultural fires ~40% weaker than wildfires |

---

### Execution Order
1. Run `task1.ipynb` for initial fire point filtering and visualization.  
2. Run `task2.ipynb` for crop maturity alignment and agricultural attribution.  
3. Run `task3.ipynb` for trend analysis (2010–2019).  
4. Run `task4.ipynb` for machine learning classification and performance evaluation.

---
