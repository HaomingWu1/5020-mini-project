# Mini Group Project I – Investigating Agricultural Burning with Remote Sensing Data

### Course: CNGF5020  
**Group Members:** Haoming Wu, Yihao Su, Wenbo Duan, Linwei Li

---

## 1. Project Overview

This project analyzes agricultural straw burning in Heilongjiang Province using satellite remote sensing data. Agricultural burning, although officially prohibited, is still practiced and its detection remains a challenge. Using MODIS active fire data, crop maturity rasters, and other geographic layers, this study develops a workflow for identifying and classifying fire points potentially linked to agricultural activities.

The analysis spans the years **2010–2019**, aiming to assess fire spatial-temporal patterns, evaluate alignment with crop harvesting, analyze long-term trends, and apply machine learning to distinguish agricultural fires.

---

## 2. Repository Structure

```
├── task1.ipynb        # Extract MODIS fire points and visualize seasonal patterns
├── task2.ipynb        # Match fire dates to crop maturity DOY from raster
├── task3.ipynb        # Analyze annual trends of county-level fire activity
├── task4.ipynb        # Train classifier to label agricultural vs. non-agricultural fire points
├── data/              # Raw data files
└── README.md
```

---

## 3. Data Sources

- **Satellite Fire Data:**  
  modis_2010_China.csv(1 km; 4 times per day). Attributes include latitude, longitude, time, Fire Radiative Power (FRP), etc. Spatiotemporal coverage: China, 2010–2019.

- **Cropland distribution and phenological data:**  
  Heilongjiang_Maize_MA_2010.tif and Heilongjiang_Wheat_MA_2010.tif. Raster datasets of maize and wheat maturity DOY (Day of Year) in Heilongjiang Province from 2010 to 2019, with a spatial resolution of 1 km. (Raster values represent the DOY of crop maturity; pixels with NoData indicate non-maize or non-wheat areas.)

- **County-level administrative boundaries of China:**  
  CHN_County.shp, used for identifying the administrative regions of wildfire points.

- **Fengyun Fire Monitoring (2016–2017):**  
  straw burning fire point monitoring data.xslx (August 2016 – February 2017). This dataset was validated through field surveys and cross-comparison with other satellite observations, providing relatively high accuracy and serving as a reference for classification (note: the dataset does not specify the crop type associated with the straw burning).

---
## 4. Data Preprocessing (Integrated in Each Task)

Unlike centralized preprocessing pipelines, this project adopted a task-driven preprocessing strategy. Each dataset was cleaned and processed within the context of the specific analytical objective. Below we summarize key data preprocessing steps as applied across Tasks 1–4.

### Task 1 – Spatiotemporal Pattern Analysis
- **Data Source:** MODIS Active Fire Data (2010–2019)
- **Processing Steps:**
  - Multiple yearly CSVs were loaded and concatenated.
  - Hold "latitude, longitude, acq_date, confidence"
  - Fire points were filtered to the spatial bounds of Heilongjiang (121–135°E, 43.5–53.5°N).
  - Derived temporal fields included: `year`, `doy` (day of year), `week`.

### Task 2 + Task4 – Crop Maturity Alignment + Classification
- **Data Sources:** MODIS Fire Data + FY (Fengyun) satellite-based straw burning fire point monitoring data
- **Processing Steps(MODIS Fire Data):**
  - Keep only the required fields (e.g., latitude, longitude, date, fire intensity, etc.)
  - Process time format: Convert time (e.g., "1230") to "12:30"
  - Filter: Retain fire points within Heilongjiang Province (43-53 degrees latitude, 121-135 degrees longitude)
  - Further filter: FRP (fire intensity)> 0, and essential data including latitude, longitude, and date are complete.
  - Merge dates and times into standard time format (e.g., "2010-05-01 12:30")
- **Processing Steps(FY satellite-based straw burning fire point monitoring data):**
  - Remove records with missing latitude and longitude (data without location information is useless)
  - Filter out fire points within Heilongjiang Province, excluding invalid data such as records without location (missing latitude and longitude) or outside the province, to ensure the subsequent analysis focuses on the target area.
  - Use gpd.GeoDataFrame to convert regular table data into "geographic data", so that each fire point has spatial location information with latitude and longitude.

### Task 3 – Trend and Change Analysis
- **Data Sources:** Heilongjiang_Maize_MA_2010.tif & Wheat_2010.tif
- **Processing Steps:**
  - Manually set the longitude and latitude bounding boxes for 13 prefecture-level cities
  

---
## 5. Methodology and Results

### Task 1 – Spatiotemporal Pattern Analysis

**Objective:** Identify seasonal distribution of fire points in Heilongjiang.

**Method:**
- Loaded and concatenated MODIS CSV files for 2010–2019
- Filtered records with confidence values indicating valid detections
- Converted `acq_date` and `acq_time` into Python datetime format
- Extracted fire week number and Day of Year (DOY)
- Selected fire points within the administrative boundary of Heilongjiang
- Aggregated weekly fire counts to examine seasonal dynamics
- Plotted bar charts showing total weekly fire occurrences

**Findings:**
- Fire activity peaks between **weeks 39–41**, coinciding with harvest season
- Major hotspots in **Harbin**, **Daqing**, **Qiqihar**, and **Sanjiang Plain**

---

### Task 2 – Crop Maturity Alignment

**Objective:** Link fire occurrences to crop harvesting time.

**Method:**
- The function defines three input parameters (fire point geographic data, crop raster file path, and crop type). The core approach is ** "year-to-year correspondence" **: First, identify the crop data for a specific year, then locate the corresponding fire points. Next, calculate the time difference between each fire point and the corresponding crop maturity date. Finally, merge the results across all years.
- Our function primarily addresses the identification of fire points associated with straw burning. By correlating "fire point locations" with "crop cultivation zones and maturity periods," and calculating temporal discrepancies, we can filter out "fire points occurring shortly after crop maturity." These fire points are high-probability candidates for straw burning. Without this function, fire point data and crop data would remain two isolated "information silos," making correlation analysis impossible.
- Analyze the distribution pattern of the time difference between fire point occurrence and crop maturity  
- Compute **`days_after_crop`** (difference between fire date and crop maturity).    
- Focus on fires occurring **30 days before to 60 days after crop maturity**.  
- Calculate indicators:  
  - Mean/median of `days_after_crop`  
  - Proportion of fires within **1–30 days after maturity** (key straw-burning indicator)  
  - Visualize using histograms to show timing distribution. 

**Findings:**
- Straw burning usually occurs within 1 to 30 days after the harvest of crops (when they are mature) (farmers will deal with the straw after the harvest). The higher this proportion, the stronger the correlation between the fire point and the burning of the crop straw.
- For wheat, **48.47%** of fire points occurred within 1–30 days after crop maturity (avg: 28.4 days)
- For maize, only **17.19%** fell in that window (avg: 39.1 days), suggesting weaker agricultural correlation

---

### Task 3 – Trend and Change Analysis

**Objective:** Analyze yearly trend in fire occurrence at the county level.

**Method:**
- County Mapping: Used raw numpy to read crop raster files (TIFF), applied hardcoded GeoTransform parameters and bounding boxes to extract county-specific pixels — no use of geospatial libraries
- Burn Rate Calculation: Standardized fire intensity across counties using the formula:
  - burn_rate = yearly fire points ÷ crop pixels per county × 1000(expressed as fires per 1,000 pixels, to eliminate area bias)
- Trend Analysis:
  - Applied Theil-Sen slope estimator to measure trend direction
  - Used Mann-Kendall test to assess significance (α = 0.05)
- Breakpoint Detection: Identified year of greatest drop in residuals from the province-wide annual fire trend line, used as a simple turning point estimate
- Visualization:
  - Provincial fire trend line plotted with a red dashed line indicating the detected breakpoint year.

**Findings:**
- Apparent decline after **2017**, especially in Harbin and Qiqihar
- Mean FRP decreased by 22%, suggesting reduced intensity


---

### Task 4 – Classification

**Objective:** Classify fire points as agricultural or non-agricultural.

**Method:**
- Seven features were derived to represent fire characteristics
  - DOY, hour, latitude, longitude, FRP, in_maize, in_wheat etc.
- Inform the model in advance about which fire points are agricultural fires, enabling it to learn the corresponding features. Agricultural fire labels were assigned based on whether a fire point occurred within 60 days after the crop maturity date 
- Split dataset into training and test sets (typically 70/30 split)
- Standardization features. The numerical ranges of different features vary significantly (for example, hour ranges from 0 to 23, while frp ranges from 0 to 1000). Standardization to the same order of magnitude is required
- Trained a tree-based classifier (Gradient Boosting) and Use the remaining 30% of the data for testing
- Predict all fire points —— Provide the final classification result

**Findings:** 
- Most important features: FRP and Intra-day time distribution
- Agricultural fires tend to have lower FRP (avg: 6.4 MW vs 15.75 MW) and peak during daytime (around 2:00), consistent with human straw-burning activity.
- In contrast, non-agricultural fires show higher intensity and tend to peak around 4:00 AM.


---

## 6. Summary Table

| Core Task                        | Key Result                                                                 |
|----------------------------------|----------------------------------------------------------------------------|
| 1. What are the dominant spatiotemporal patterns of fire activity in the study region?| Fire peaks in **weeks 39–41 (Sept–Oct)**. Hotspots in **Harbin, Qiqihar, Sanjiang Plain** |
| 2. To what extent can the observed fire hotspots be attributed to the post-harvest 
burning of corn and wheat?       | Wheat: 48.47% classified as agricultural fires; Maize: 17.19%  |
| 3. What are the long-term spatiotemporal trends of fire activity (2010–2019)? | After **2017**, most counties show a sharp decline in fire density  |
| 4. Are there distinct characteristics that differentiate agricultural fires from other 
fires?    | agricultural fires show lower FRP and peak in daytime (2:00) |

---

## 7. Execution Instructions

1. Ensure the raw data files are prepared
2. Place them in the `/data/` folder or adjust paths in notebooks
3. Run notebooks in order: `task1 → task2 → task3 → task4`
4. Install required packages using pip:

```bash
pip install pandas geopandas numpy rasterio matplotlib scikit-learn
```
## 8. Methodological Limitations and Uncertainty(Challenge question 2): 

### Limitations of Methodology

While our classification of agricultural fires is grounded in spatial and temporal alignment with crop maturity, several sources of uncertainty affect the robustness of our results:

- **Raster Resolution**  
  The crop maturity maps have a coarse resolution (~1 km). Fire points, which are point-based detections from MODIS (approx. 1 km²), may fall into mixed land cover pixels, introducing misclassification risk.

- **Fixed Crop Calendar Assumption**  
  We applied a fixed DOY (Day of Year) raster for each crop type across all years (2010–2019). This ignores annual variability in planting/harvest times due to climate or farming practices, possibly misaligning fire and crop maturity periods.

- **Binary Crop Presence (in_maize / in_wheat)**  
  The model simplifies crop detection by assigning a binary label based on raster overlay, without accounting for crop density, rotation, or intercropping.

- **FY Reference Dataset Limitation**  
  The Fengyun fire monitoring dataset only covers a short period (Aug 2016 – Feb 2017) and cannot provide validation for other years, limiting the reliability of model training labels.

- **Spatial Masking Assumptions**  
  The crop raster files are not matched with actual cropland boundaries or land-use masks, so fires occurring in non-cropland areas may be mistakenly linked to agriculture.

### Impact on Results

These limitations can affect the accuracy of agricultural fire classification, especially in edge pixels or transition zones between land use types.  
For instance, wheat fires show stronger alignment (48.47%) because wheat is mostly planted in concentrated zones, while maize fields may be more dispersed or harder to delineate in the raster.

### Suggested Improvements

**Recommended Additional Dataset:**  
**Land Use / Land Cover (LULC) Data**

**Justification:**  
Incorporating high-resolution LULC data would allow us to mask non-cropland areas (e.g., forest, urban), improving the precision of fire–crop overlays. Fires outside agricultural zones could be excluded with higher confidence.

**Implementation:**  
Apply the LULC layer as a spatial filter before calculating `days_after_crop` or training the classifier.  
This would reduce noise from irrelevant fire points and enhance the model’s focus on genuine agricultural burning.

