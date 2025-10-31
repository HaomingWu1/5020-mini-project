# Mini Group Project I – Investigating Agricultural Burning with Remote Sensing Data

### Course: CNGF5020  
**Group Members:** [Fill in Names]

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
├── data/              # Raw data files: fire CSVs, shapefiles, crop rasters
└── README.md
```

---

## 3. Data Sources

- **MODIS Active Fire Detections (2010–2019):**  
  Latitude, longitude, acquisition date/time, FRP, confidence

- **Cropland distribution and phenological data:**  
  Raster images showing the DOY when crops mature

- **Administrative Boundaries:**  
  County-level shapefiles of Heilongjiang

- **Fengyun Fire Monitoring (2016–2017):**  
  FY reference dataset used for partial validation

---

## 4. Methodology and Results

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
- Loaded raster files representing crop maturity (DOY)
- For each fire point, used rasterio to extract crop maturity DOY from pixel value
- Computed Δdays as the difference between fire DOY and crop DOY
- Created histograms and scatter plots to show distribution of Δdays
- No explicit thresholding was applied (e.g., ±30), but analysis focused on temporal alignment

**Findings:**
- Straw burning usually occurs within 1 to 30 days after the harvest of crops (when they are mature) (farmers will deal with the straw after the harvest). The higher this proportion, the stronger the correlation between the fire point and the burning of the crop straw.
- For wheat, **48.47%** of fire points occurred within 1–30 days after crop maturity (avg: 28.4 days)
- For maize, only **17.19%** fell in that window (avg: 39.1 days), suggesting weaker agricultural correlation

---

### Task 3 – Trend and Change Analysis

**Objective:** Analyze yearly trend in fire occurrence at the county level.

**Method:**
- Merged fire point locations with county boundary shapefile to assign county names
- For each year and county, counted number of fire points
- Loaded binary cropland raster map to determine total number of cropland pixels per county
- Normalized yearly fire counts by cropland area (fire density)
- Plotted time series of county fire densities over 10 years
- No statistical tests (Mann–Kendall or Theil–Sen) applied, only visual trend analysis

**Findings:**
- Apparent decline after **2017**, especially in Harbin and Qiqihar
- 2017 marked a province-wide drop (>60%) consistent with intensified enforcement of open burning bans
- Mean FRP decreased by 22%, suggesting reduced intensity


---

### Task 4 – Classification

**Objective:** Classify fire points as agricultural or non-agricultural.

**Method:**
- Constructed feature matrix from MODIS fire attributes and spatial overlay:
  - DOY, hour, latitude, longitude, FRP, raster-based crop type, etc.
- Label assignment was done using spatial and temporal logic (e.g., fire in cropland + near harvest)
- Split dataset into training and test sets (typically 70/30 split)
- Trained a tree-based classifier (Gradient Boosting)
- Evaluated model using accuracy, precision, recall metrics on test set

**Findings:**
- Classifier achieved high performance 
- Most important features:  FRP and Intra-day time distribution
- Agricultural fires tend to have lower FRP (avg: 6.4 MW vs 15.75 MW) and peak during daytime (around 2:00), consistent with human straw-burning activity.
- In contrast, non-agricultural fires show higher intensity and tend to peak around 4:00 AM.


---

## 6. Summary Table

| Core Task                        | Key Result                                                                 |
|----------------------------------|----------------------------------------------------------------------------|
| 1. When and where do fires occur?| Fire peaks in **weeks 39–41 (Sept–Oct)**. Hotspots in **Harbin, Qiqihar, Sanjiang Plain** |
| 2. Are fires agricultural?       | Wheat: 48.47% classified as agricultural fires; Maize: 17.19%  |
| 3. Are agricultural fires changing over time? | After **2017**, most counties show a sharp decline in fire density  |
| 4. Can we predict fire types?    | agricultural fires show lower FRP and peak in daytime (2:00) |

---

## 7. Execution Instructions

1. Ensure the raw data files are prepared
2. Place them in the `/data/` folder or adjust paths in notebooks
3. Run notebooks in order: `task1 → task2 → task3 → task4`
4. Install required packages using pip:

```bash
pip install pandas geopandas numpy rasterio matplotlib scikit-learn
```

