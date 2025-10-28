# 🛰️ Mini Group Project I – Investigating Agricultural Burning with Remote Sensing Data

 
### **Group Members:**  
 

---

## Overview  

This project investigates **agricultural residue burning** in Heilongjiang Province using multi-source remote sensing data. The analysis integrates **MODIS fire detection**, **FY straw-burning monitoring data**, and **crop phenology datasets** to identify and classify agricultural burning events.  

Two key Jupyter notebooks (`q2_cngf5020.ipynb` and `q4_cngf5020.ipynb`) perform distinct but complementary functions:  
- **Q2:** Data cleaning, spatial filtering, and temporal matching between fire events and crop maturity dates.  
- **Q4:** Machine learning classification of agricultural versus non-agricultural fires using Gradient Boosting.  

The results contribute to understanding **where and when straw burning occurs**, and how machine learning can improve automated fire-type classification.

---

## Repository Structure  

```
├── q2_cngf5020.ipynb      # Data preprocessing and spatiotemporal analysis
├── q4_cngf5020.ipynb      # Gradient Boosting fire classification
├── data/
│   ├── modis_2010_China.csv
│   ├── Heilongjiang_Maize_MA_2010.tif
│   ├── Heilongjiang_Wheat_MA_2010.tif
│   ├── straw burning fire point monitoring data.xlsx
│   └── CHN_County.shp
└── README.md              # Final project report
```

---

## Objectives  

1. Clean and preprocess satellite fire data within Heilongjiang Province.  
2. Correlate fire occurrences with crop maturity timing to identify likely **agricultural fires**.  
3. Build a **machine learning classifier** to distinguish agricultural vs. non-agricultural fires.  
4. Evaluate fire characteristics (intensity, timing, spatial distribution) and generate visual insights.

---

## Part 2: `q2_cngf5020.ipynb` – Data Cleaning and Temporal-Spatial Analysis  

### **Purpose**  
Prepare a reliable dataset for identifying possible straw-burning fires by cleaning satellite and field-verified data, and linking fire events to crop phenology information.

### **Key Steps**

1. **Define Study Area**  
   - Load China’s county-level shapefile (`CHN_County.shp`).  
   - Filter boundary to **Heilongjiang Province** for targeted analysis.  

2. **MODIS Fire Data Cleaning**  
   - Load `modis_2010_China.csv` and keep only essential fields (latitude, longitude, date, FRP).  
   - Convert time field (e.g., `"1230"`) to `"12:30"` using `datetime`.  
   - Remove incomplete or invalid records.  
   - Filter to Heilongjiang’s coordinates (lat 43–53°N, lon 121–135°E).  
   - Merge date and time into a single `datetime` column.  

3. **FY Straw Burning Data Processing**  
   - Load `straw burning fire point monitoring data.xlsx`.  
   - Drop missing coordinates and filter to Heilongjiang’s region.  
   - Convert to `GeoDataFrame` for spatial operations.  

4. **Crop Phenology Matching**  
   - Match yearly fire data to corresponding maize and wheat maturity rasters (`Heilongjiang_Maize_MA_YYYY.tif`, etc.).  
   - Compute **`days_after_crop`** (difference between fire date and crop maturity).  

5. **Temporal Filtering and Statistical Analysis**  
   - Focus on fires occurring **30 days before to 60 days after crop maturity**.  
   - Calculate indicators:  
     - Mean/median of `days_after_crop`  
     - Proportion of fires within **1–30 days after maturity** (key straw-burning indicator)  
   - Visualize using histograms to show timing distribution.  

### **Outputs**  
- Clean dataset of fire events with `days_after_crop` attribute.  
- Visualizations of fire timing relative to crop maturity.  
- Summary statistics supporting classification logic.  

### **Core Packages**  
`pandas`, `numpy`, `geopandas`, `matplotlib`, `rasterio`, `datetime`

---

## Part 4: `q4_cngf5020.ipynb` – Machine Learning Classification  

### **Purpose**  
Develop a **Gradient Boosting model** to automatically classify fire events as agricultural or non-agricultural based on their spatial and temporal characteristics.

### **Key Steps**

1. **Feature Engineering**  
   Seven features were derived to represent fire characteristics:  
   - Day of Year (DOY)  
   - Fire Radiative Power (FRP)  
   - Latitude and Longitude  
   - Hour of occurrence  
   - Whether the fire is in maize area (`in_maize`)  
   - Whether in wheat area (`in_wheat`)  
   - Whether within post-harvest window (`in_harvest_window`)  

2. **Label Definition**  
   - Fires located within crop zones and within **14 days post-harvest** are labeled as **agricultural (1)**.  
   - Others are labeled as **non-agricultural (0)**.  

3. **Data Splitting and Normalization**  
   - 70% training set, 30% testing set.  
   - Standardize features to equalize scale (e.g., FRP vs hour).  

4. **Model Training – Gradient Boosting Classifier**  
   - Trains 100 decision trees (`n_estimators=100`).  
   - Each new tree corrects the errors of the previous ensemble.  
   - Produces final predictions via weighted voting.  

5. **Evaluation and Visualization**  
   - Compute accuracy, precision, recall, and confusion matrix.  
   - Visualize feature importance to identify top predictors (e.g., `days_after_crop`, FRP).  

6. **Prediction and Mapping**  
   - Apply model to classify all fire points in Heilongjiang.  
   - Generate visual maps showing predicted agricultural fire clusters.  

### **Outputs**  
- Trained model (`GradientBoostingClassifier`).  
- Classification results with predicted labels.  
- Performance metrics and feature-importance visualization.  

### **Core Packages**  
`pandas`, `numpy`, `scikit-learn`, `matplotlib`, `geopandas`, `seaborn`

---

## Summary of Insights  

- A large proportion of fire events occur **within 1–30 days after crop maturity**, indicating strong temporal linkage to straw burning.  
- Agricultural fires tend to have **lower FRP values** compared to wildfires, suggesting smaller-scale, controlled burns.  
- The machine learning model effectively replicates expert judgment logic, achieving high classification accuracy and interpretability.  

---

## Environment & Dependencies  

```bash
Python >= 3.9
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

- Crop maturity data may contain uncertainty (e.g., fixed DOY thresholds).  
- Satellite detection frequency may miss short-duration fires.  
- Future improvements could incorporate meteorological factors (wind, humidity) or pollution data (PM2.5) to strengthen environmental interpretation.  
