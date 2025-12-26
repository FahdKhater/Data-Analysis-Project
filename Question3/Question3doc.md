# Question 3: Seasonal Tourism Patterns - Comprehensive Documentation

## Research Question
**Are certain regions more affected by seasonal tourism trends than others?**

---

## Table of Contents
1. [Overview](#overview)
2. [Data Preprocessing](#data-preprocessing)
3. [Data Structures Created](#data-structures-created)
4. [Metrics and Indexes](#metrics-and-indexes)
5. [Exploratory Data Analysis](#exploratory-data-analysis)
6. [Visualizations](#visualizations)
7. [Key Findings](#key-findings)

---

## Overview

This analysis examines seasonal tourism patterns across 14 cities spanning 6 continents to identify which regions experience the most significant seasonal variations in tourism activity. The analysis uses Google Trends data representing normalized tourism interest (tourism index) from December 2023 to December 2025.

**Dataset Characteristics:**
- **Source**: `TouristArrival/trends.csv`
- **Original Format**: Wide format with 104 rows (weekly dates) × 15 columns (date + 14 cities)
- **Date Range**: 2023-12-31 to 2025-12-21 (104 weeks)
- **Cities Analyzed**: 14 cities across 6 continents
- **Final Data Points**: 1,456 records (104 dates × 14 cities)

---

## Data Preprocessing

### Step 1: Data Loading and Initial Inspection

**Input**: `../TouristArrival/trends.csv`
- **Shape**: (104, 15)
- **Columns**: `date` + 14 city columns (Paris, Barcelona, Tokyo, New York, London, Rome, Amsterdam, Sydney, Bangkok, Istanbul, Cairo, Rio de Janeiro, Venice, Los Angeles)
- **Data Type**: Tourism index values (normalized 0-1 scale)

**Actions Performed**:
1. Loaded CSV file into pandas DataFrame
2. Verified data completeness (no missing values)
3. Checked data statistics (mean, std, min, max for each city)
4. Confirmed tourism index values are within [0, 1] range

**Output**: `df_trends` - Original wide format dataset

---

### Step 2: Data Reshaping (Wide to Long Format)

**Transformation Purpose**: Convert from wide format (one column per city) to long format (one row per city-date combination) to enable time-series analysis and grouping operations.

**Process**:
```python
# Used pandas melt() function to reshape
df_long = pd.melt(
    df_trends,
    id_vars=['date'],
    value_vars=city_columns,  # All 14 city columns
    var_name='city',
    value_name='tourism_index'
)
```

**Result**:
- **Input Shape**: (104, 15)
- **Output Shape**: (1,456, 3) - 104 dates × 14 cities
- **Columns**: `date`, `city`, `tourism_index`

**Why This Format?**
- Enables grouping by city, continent, or temporal features
- Facilitates time-series analysis
- Required for aggregation operations (monthly, quarterly averages)

---

### Step 3: Temporal Feature Engineering

**Purpose**: Extract temporal components from dates to enable seasonal analysis.

**Features Created**:
1. **`year`**: Year component (2023, 2024, 2025)
2. **`month`**: Month number (1-12)
3. **`quarter`**: Quarter number (1-4)
   - Q1: Jan-Mar, Q2: Apr-Jun, Q3: Jul-Sep, Q4: Oct-Dec
4. **`day_of_year`**: Day number within year (1-365/366)
5. **`week_of_year`**: ISO week number (1-52/53)

**Implementation**:
```python
df_long['year'] = df_long['date'].dt.year
df_long['month'] = df_long['date'].dt.month
df_long['quarter'] = df_long['date'].dt.quarter
df_long['day_of_year'] = df_long['date'].dt.dayofyear
df_long['week_of_year'] = df_long['date'].dt.isocalendar().week
```

**Final Structure**: `df_long` now has 9 columns:
- `date`, `city`, `tourism_index`, `year`, `month`, `quarter`, `day_of_year`, `week_of_year`, `continent`

---

### Step 4: Geographic Categorization

**Purpose**: Group cities by continent to enable regional analysis.

**City-to-Continent Mapping**:
- **Europe** (6 cities): Paris, Barcelona, London, Rome, Amsterdam, Venice
- **Asia** (3 cities): Tokyo, Bangkok, Istanbul
- **North America** (2 cities): New York, Los Angeles
- **South America** (1 city): Rio de Janeiro
- **Africa** (1 city): Cairo
- **Oceania** (1 city): Sydney

**Implementation**:
```python
city_to_continent = {
    'Paris': 'Europe',
    'Barcelona': 'Europe',
    # ... (complete mapping)
}
df_long['continent'] = df_long['city'].map(city_to_continent)
```

**Result**: Each record now has a continent label for regional aggregation.

---

### Step 5: Data Quality Checks

**Checks Performed**:
1. **Missing Values**: Verified no missing values in any column
2. **Value Range**: Confirmed tourism_index values are within [0, 1]
3. **Data Completeness**: Verified 104 dates × 14 cities = 1,456 records
4. **Date Continuity**: Confirmed date range spans expected period

**Handling Strategy** (if missing values found):
- Forward fill within each city's time series
- Backward fill for any remaining gaps
- (No missing values were found in this dataset)

---

### Step 6: Data Aggregation and Splitting

#### 6.1 Continent-Level Monthly Averages

**Purpose**: Create aggregated monthly data for continent-level analysis.

**Process**:
```python
monthly_avg = continent_df.groupby(['year', 'month', 'continent'])['tourism_index'].mean().reset_index()
```

**Output**: `df_continent_monthly`
- **Shape**: (150, 5) - 6 continents × ~25 months
- **Columns**: `year`, `month`, `continent`, `tourism_index`, `date`
- **Use Case**: Continent-level seasonal pattern analysis

#### 6.2 City-Level Monthly Averages

**Purpose**: Create city-specific monthly data for detailed analysis.

**Process**:
```python
city_monthly_avg = df_long.groupby(['city', 'continent', 'year', 'month'])['tourism_index'].mean().reset_index()
```

**Output**: `city_monthly_avg`
- **Shape**: (350, 6) - 14 cities × ~25 months
- **Columns**: `city`, `continent`, `year`, `month`, `tourism_index`, `date`
- **Use Case**: City-level seasonal pattern analysis

#### 6.3 Continent-Specific Datasets

**Purpose**: Split data by continent for individual analysis.

**Output**: 6 separate CSV files:
- `continent_africa.csv`
- `continent_asia.csv`
- `continent_europe.csv`
- `continent_north_america.csv`
- `continent_oceania.csv`
- `continent_south_america.csv`

Each contains all records for cities in that continent.

---

## Data Structures Created

### 1. `df_long` (Long Format Dataset)

**Purpose**: Primary dataset in long format for all analyses.

**Structure**:
- **Shape**: (1,456, 9)
- **Columns**:
  - `date`: Date of observation (datetime)
  - `city`: City name (string)
  - `tourism_index`: Normalized tourism interest value (float, 0-1)
  - `year`: Year (int)
  - `month`: Month number (int, 1-12)
  - `quarter`: Quarter number (int, 1-4)
  - `day_of_year`: Day of year (int, 1-365)
  - `week_of_year`: Week of year (int, 1-52)
  - `continent`: Continent name (string)

**Use Cases**:
- Time-series plotting
- Statistical calculations
- Grouping and aggregation
- Filtering by city, continent, or time period

**Saved As**: `Question3/preprocessed_data/df_long.csv`

---

### 2. `df_continent_monthly` (Continent Monthly Averages)

**Purpose**: Monthly aggregated tourism index by continent.

**Structure**:
- **Shape**: (150, 5)
- **Columns**:
  - `year`: Year (int)
  - `month`: Month number (int, 1-12)
  - `continent`: Continent name (string)
  - `tourism_index`: Average tourism index for that continent-month (float)
  - `date`: First day of month (datetime)

**Calculation**: Average of all cities in a continent for each month-year combination.

**Use Cases**:
- Continent-level seasonal pattern visualization
- Comparing seasonal trends across regions
- Identifying peak and low seasons by continent

**Saved As**: `Question3/preprocessed_data/df_continent_monthly.csv`

---

### 3. `city_monthly_avg` (City Monthly Averages)

**Purpose**: Monthly aggregated tourism index by individual city.

**Structure**:
- **Shape**: (350, 6)
- **Columns**:
  - `city`: City name (string)
  - `continent`: Continent name (string)
  - `year`: Year (int)
  - `month`: Month number (int, 1-12)
  - `tourism_index`: Average tourism index for that city-month (float)
  - `date`: First day of month (datetime)

**Calculation**: Average of all weekly values for a city in each month-year combination.

**Use Cases**:
- City-level seasonal analysis
- Identifying city-specific patterns
- Comparing cities within the same continent

**Saved As**: `Question3/preprocessed_data/city_monthly_avg.csv`

---

### 4. `df_seasonal_stats` (Seasonal Statistics Summary)

**Purpose**: Statistical summary of seasonal variation by continent.

**Structure**:
- **Shape**: (6, 11) - One row per continent
- **Columns**:
  - `continent`: Continent name (string)
  - `num_cities`: Number of cities in continent (int)
  - `mean_trend`: Mean tourism index across all data (float)
  - `std_trend`: Standard deviation of tourism index (float)
  - `min_trend`: Minimum tourism index value (float)
  - `max_trend`: Maximum tourism index value (float)
  - `trend_range`: max_trend - min_trend (float)
  - `coefficient_of_variation`: (std_trend / mean_trend) × 100 (float, %)
  - `monthly_range`: Maximum monthly average - Minimum monthly average (float)
  - `monthly_std_mean`: Average of monthly standard deviations (float)
  - `quarterly_range`: Maximum quarterly average - Minimum quarterly average (float)

**Use Cases**:
- Quick comparison of variability across continents
- Identifying most/least variable regions
- Statistical summary for reporting

**Saved As**: `Question3/preprocessed_data/df_seasonal_stats.csv`

---

### 5. `df_seasonal_variation` (Seasonal Variation Metrics)

**Purpose**: Detailed metrics quantifying seasonal variation patterns.

**Structure**:
- **Shape**: (6, 9) - One row per continent
- **Columns**:
  - `continent`: Continent name (string)
  - `avg_monthly_cv`: Average coefficient of variation across months (float, %)
  - `max_monthly_cv`: Maximum coefficient of variation across months (float, %)
  - `peak_season_month`: Month number with highest average tourism index (int, 1-12)
  - `low_season_month`: Month number with lowest average tourism index (int, 1-12)
  - `peak_value`: Tourism index value during peak season (float)
  - `low_value`: Tourism index value during low season (float)
  - `seasonal_amplitude`: peak_value - low_value (float)
  - `seasonal_amplitude_pct`: ((peak_value - low_value) / low_value) × 100 (float, %)

**Use Cases**:
- Identifying peak and low seasons
- Quantifying seasonal variation magnitude
- Ranking regions by seasonal sensitivity

**Saved As**: `Question3/preprocessed_data/df_seasonal_variation.csv`

---

### 6. `continent_data` (Dictionary of Continent-Specific DataFrames)

**Purpose**: Separate datasets for each continent for individual analysis.

**Structure**: Dictionary with 6 keys (continent names), each containing:
- **Shape**: Varies by continent (104-624 rows)
- **Columns**: Same as `df_long` (9 columns)

**Contents**:
- `continent_data['Africa']`: 104 rows (1 city × 104 dates)
- `continent_data['Asia']`: 312 rows (3 cities × 104 dates)
- `continent_data['Europe']`: 624 rows (6 cities × 104 dates)
- `continent_data['North America']`: 208 rows (2 cities × 104 dates)
- `continent_data['Oceania']`: 104 rows (1 city × 104 dates)
- `continent_data['South America']`: 104 rows (1 city × 104 dates)

**Saved As**: Individual CSV files in `Question3/preprocessed_data/`

---

## Metrics and Indexes

### 1. Tourism Index

**Definition**: Normalized measure of tourism interest/activity, derived from Google Trends data.

**Scale**: 0 to 1 (normalized)
- **0**: Minimum interest/activity
- **1**: Maximum interest/activity (peak)

**Interpretation**:
- Higher values indicate greater tourism interest/activity
- Values are relative, not absolute counts
- Enables comparison across cities and time periods

**Calculation**: Provided by Google Trends API (normalized to 0-1 scale)

---

### 2. Coefficient of Variation (CV)

**Definition**: Ratio of standard deviation to mean, expressed as a percentage. Measures relative variability.

**Formula**:
```
CV = (Standard Deviation / Mean) × 100
```

**Interpretation**:
- **Low CV (< 20%)**: Low variability, stable tourism levels
- **Medium CV (20-40%)**: Moderate variability, some seasonal patterns
- **High CV (> 40%)**: High variability, strong seasonal patterns

**Why Use CV?**
- Normalizes variability by mean, enabling comparison across regions with different baseline tourism levels
- A region with mean=0.3 and std=0.15 has same CV as region with mean=0.6 and std=0.30

**In This Analysis**:
- `coefficient_of_variation`: Overall CV for entire time period
- `avg_monthly_cv`: Average of monthly CVs (measures month-to-month variability)
- `max_monthly_cv`: Maximum monthly CV (identifies most variable month)

**Example**: Africa has CV = 58.42%, indicating high variability relative to its mean tourism index.

---

### 3. Seasonal Amplitude

**Definition**: Absolute difference between peak season and low season tourism index values.

**Formula**:
```
Seasonal Amplitude = Peak Season Value - Low Season Value
```

**Interpretation**:
- **Large Amplitude**: Significant difference between peak and low seasons (highly seasonal)
- **Small Amplitude**: Similar tourism levels year-round (less seasonal)

**Units**: Same as tourism index (0-1 scale)

**Example**: Africa has amplitude = 0.389, meaning peak season is 0.389 points higher than low season.

---

### 4. Seasonal Amplitude Percentage

**Definition**: Percentage change from low season to peak season, relative to low season value.

**Formula**:
```
Seasonal Amplitude % = ((Peak Value - Low Value) / Low Value) × 100
```

**Interpretation**:
- **High Percentage**: Large relative increase from low to peak season
- **Low Percentage**: Small relative increase from low to peak season

**Why Use Percentage?**
- Normalizes amplitude by low season value
- Enables comparison across regions with different baseline levels
- Shows relative impact of seasonality

**Example**: Africa has amplitude % = 183.5%, meaning peak season is 183.5% higher than low season (nearly 3x the low season value).

---

### 5. Standard Deviation

**Definition**: Measure of spread/dispersion of tourism index values around the mean.

**Formula**:
```
σ = √(Σ(xi - μ)² / N)
```

**Interpretation**:
- **High Standard Deviation**: Wide spread of values (high variability)
- **Low Standard Deviation**: Narrow spread of values (low variability)

**In This Analysis**:
- `std_trend`: Overall standard deviation for entire time period
- `monthly_std_mean`: Average of monthly standard deviations

**Example**: Africa has std = 0.210, indicating wide variation in tourism index values.

---

### 6. Range

**Definition**: Difference between maximum and minimum values.

**Formula**:
```
Range = Maximum Value - Minimum Value
```

**Interpretation**:
- **Large Range**: Wide spread from minimum to maximum
- **Small Range**: Narrow spread from minimum to maximum

**In This Analysis**:
- `trend_range`: Overall range (max - min) for entire time period
- `monthly_range`: Range of monthly averages (max monthly avg - min monthly avg)
- `quarterly_range`: Range of quarterly averages

**Example**: South America has range = 0.89, meaning tourism index spans from 0.11 to 1.00.

---

### 7. Peak Season Month

**Definition**: Month number (1-12) with the highest average tourism index.

**Calculation**:
```python
monthly_avg = continent_df.groupby('month')['tourism_index'].mean()
peak_month = monthly_avg.idxmax()
```

**Interpretation**:
- Identifies when tourism peaks occur
- Reveals seasonal patterns (e.g., summer peaks, winter peaks)
- Enables comparison of peak timing across regions

**Example**: Africa's peak season is November (month 11), while Europe's peak is August (month 8).

---

### 8. Low Season Month

**Definition**: Month number (1-12) with the lowest average tourism index.

**Calculation**:
```python
monthly_avg = continent_df.groupby('month')['tourism_index'].mean()
low_month = monthly_avg.idxmin()
```

**Interpretation**:
- Identifies when tourism dips occur
- Reveals seasonal patterns (e.g., winter lows, summer lows)
- Enables comparison of low season timing across regions

**Example**: Africa's low season is June (month 6), while Europe's low is December (month 12).

---

## Exploratory Data Analysis

### Analysis Approach

The EDA process follows these steps:
1. **Load Preprocessed Data**: Import all datasets created in preprocessing
2. **Data Verification**: Confirm data quality and completeness
3. **Visualization**: Create multiple visualizations to explore patterns
4. **Statistical Summary**: Calculate and compare metrics
5. **Interpretation**: Draw conclusions about seasonal patterns

---

## Visualizations

### Visualization 1: Time Series Trends by Continent

**Type**: Line plot (multiple lines)

**Purpose**: Show temporal evolution of tourism index for each continent over the entire time period.

**Structure**:
- **X-axis**: Date (time)
- **Y-axis**: Tourism Index (0-1)
- **Lines**: One line per continent
- **Data**: Weekly averages by continent

**Key Insights**:
- **Wider vertical spread** = More seasonal variation
- **Steeper slopes** = Rapid changes (strong seasonality)
- **Flat lines** = Stable tourism (low seasonality)

**Findings**:
- Africa and South America show the widest fluctuations
- Oceania and Europe show more stable patterns
- Different continents have different peak timing

**Code Location**: `q3EDA.ipynb` Cell 6

---

### Visualization 2: Monthly Averages by Continent

**Type**: Line plot with markers

**Purpose**: Compare average tourism index by month across continents, revealing seasonal patterns.

**Structure**:
- **X-axis**: Month (1-12, labeled as Jan-Dec)
- **Y-axis**: Average Tourism Index
- **Lines**: One line per continent
- **Markers**: Points at each month

**Key Insights**:
- **Steeper curves** = Larger seasonal variation
- **Peak months** = Highest points on each line
- **Low months** = Lowest points on each line
- **Parallel lines** = Similar seasonal patterns

**Findings**:
- Most continents peak in August (summer)
- Africa and Oceania peak in November
- Different continents have different seasonal amplitudes

**Code Location**: `q3EDA.ipynb` Cell 8

---

### Visualization 3: Seasonal Variation Metrics Comparison

**Type**: 2×2 grid of horizontal bar charts

**Purpose**: Compare four key metrics of seasonal variation across continents.

**Subplots**:

1. **Coefficient of Variation**
   - **Metric**: Average monthly CV (%)
   - **Interpretation**: Higher = More variable
   - **Finding**: South America (35.8%) and Africa (32.1%) are most variable

2. **Seasonal Amplitude Percentage**
   - **Metric**: Peak-to-low season percentage difference
   - **Interpretation**: Higher = Larger seasonal swing
   - **Finding**: Africa (183.5%) and South America (165.5%) have largest swings

3. **Standard Deviation**
   - **Metric**: Spread of tourism index values
   - **Interpretation**: Higher = More spread
   - **Finding**: Africa (0.210) and South America (0.176) have highest spread

4. **Range (Peak - Low)**
   - **Metric**: Absolute difference between peak and low seasons
   - **Interpretation**: Higher = Larger absolute difference
   - **Finding**: South America (0.89) and Africa (0.85) have largest ranges

**Key Insight**: All four metrics consistently identify Africa and South America as most affected by seasonality.

**Code Location**: `q3EDA.ipynb` Cell 10

---

### Visualization 4: Heatmap of Seasonal Patterns

**Type**: Heatmap (color-coded matrix)

**Purpose**: Comprehensive view of tourism index across months (columns) and continents (rows).

**Structure**:
- **Rows**: Continents (sorted by variability)
- **Columns**: Months (Jan-Dec)
- **Colors**: 
  - **Green**: High tourism index
  - **Red**: Low tourism index
  - **Yellow**: Medium tourism index

**Key Insights**:
- **Color variation within a row** = Seasonal variation (more colors = more seasonal)
- **Consistent color in a row** = Stable tourism (less seasonal)
- **Dark green cells** = Peak months
- **Dark red cells** = Low months

**Findings**:
- Africa and South America show most color variation (most seasonal)
- Oceania shows most consistent color (least seasonal)
- August is peak month for most continents
- Different low seasons across continents

**Code Location**: `q3EDA.ipynb` Cell 12

---

### Visualization 5: Distribution of Tourism Index by Continent

**Type**: Box plot

**Purpose**: Show distribution, spread, and outliers of tourism index for each continent.

**Structure**:
- **X-axis**: Continents (sorted by variability)
- **Y-axis**: Tourism Index
- **Box Elements**:
  - **Box**: Interquartile range (IQR) - middle 50% of values
  - **Line in box**: Median
  - **Whiskers**: Range (excluding outliers)
  - **Dots**: Outliers (extreme values)
  - **Dashed line**: Mean

**Key Insights**:
- **Wider boxes** = More variability in middle 50% of values
- **Longer whiskers** = Greater range of values
- **More outliers** = More extreme seasonal variations
- **Higher mean** = Higher average tourism index

**Findings**:
- Africa and South America have widest boxes and most outliers
- Oceania has narrowest box (most stable)
- Europe has highest median tourism index

**Code Location**: `q3EDA.ipynb` Cell 14

---

### Visualization 6: Peak and Low Season Analysis

**Type**: Two-panel visualization

**Purpose**: Compare peak and low seasons across continents.

**Panel 1: Peak vs Low Values (Bar Chart)**
- **Type**: Horizontal grouped bar chart
- **X-axis**: Tourism Index
- **Y-axis**: Continents
- **Bars**: 
  - Green bars = Peak season values
  - Red bars = Low season values
- **Gap between bars** = Seasonal amplitude

**Panel 2: Peak and Low Months (Scatter Plot)**
- **Type**: Scatter plot with connecting lines
- **X-axis**: Month (1-12)
- **Y-axis**: Continents
- **Markers**:
  - Green triangles (^) = Peak season months
  - Red triangles (v) = Low season months
- **Lines**: Connect peak to low season (show seasonal cycle)

**Key Insights**:
- **Larger gaps in Panel 1** = Greater seasonal variation
- **Different peak months** = Regional seasonal differences
- **Lines in Panel 2** = Visual representation of seasonal cycle length

**Findings**:
- Africa has largest gap (0.60 peak vs 0.21 low)
- Most continents peak in August, but Africa and Oceania peak in November
- Low seasons vary: February (North America), April (South America), June (Africa/Asia), December (Europe)

**Code Location**: `q3EDA.ipynb` Cell 16

---

## Key Findings

### 1. Most Affected Regions

**By Coefficient of Variation:**
1. **South America** (35.84% CV)
   - Seasonal Amplitude: 165.5%
   - Peak: August (0.50)
   - Low: April (0.19)

2. **Africa** (32.10% CV)
   - Seasonal Amplitude: 183.5%
   - Peak: November (0.60)
   - Low: June (0.21)

3. **Europe** (25.22% CV)
   - Seasonal Amplitude: 55.6%
   - Peak: August (0.70)
   - Low: December (0.45)

**Interpretation**: These regions show high relative variability, indicating strong seasonal patterns.

---

### 2. Least Affected Regions

**By Coefficient of Variation:**
1. **Oceania** (15.72% CV)
   - Seasonal Amplitude: 51.0%
   - Peak: November (0.59)
   - Low: May (0.39)

2. **Asia** (24.76% CV)
   - Seasonal Amplitude: 76.9%
   - Peak: August (0.59)
   - Low: June (0.34)

3. **North America** (24.82% CV)
   - Seasonal Amplitude: 116.8%
   - Peak: August (0.58)
   - Low: February (0.27)

**Interpretation**: These regions show lower relative variability, indicating more stable tourism levels throughout the year.

---

### 3. Seasonal Amplitude Ranking

**From Highest to Lowest:**
1. **Africa**: 183.5% (Peak: Nov 0.60, Low: Jun 0.21)
2. **South America**: 165.5% (Peak: Aug 0.50, Low: Apr 0.19)
3. **North America**: 116.8% (Peak: Aug 0.58, Low: Feb 0.27)
4. **Asia**: 76.9% (Peak: Aug 0.59, Low: Jun 0.34)
5. **Europe**: 55.6% (Peak: Aug 0.70, Low: Dec 0.45)
6. **Oceania**: 51.0% (Peak: Nov 0.59, Low: May 0.39)

**Key Insight**: Africa and South America experience nearly 2-3x increase from low to peak season, while Oceania and Europe have more moderate increases (~50-55%).

---

### 4. Peak Season Patterns

**August Peak** (Summer in Northern Hemisphere):
- South America
- North America
- Asia
- Europe

**November Peak** (Late Fall/Early Winter):
- Africa
- Oceania

**Interpretation**: Most regions peak during summer months (August), but Africa and Oceania have different peak timing, possibly due to:
- Southern Hemisphere seasons (opposite to Northern Hemisphere)
- Regional climate patterns
- Cultural/event-driven tourism

---

### 5. Low Season Patterns

**February Low** (Winter):
- North America

**April Low** (Spring):
- South America

**May Low** (Late Spring):
- Oceania

**June Low** (Early Summer):
- Africa
- Asia

**December Low** (Winter):
- Europe

**Interpretation**: Low seasons vary significantly across regions, reflecting:
- Different climate patterns
- Regional tourism drivers
- Cultural factors

---

### 6. Regional Characteristics

**Africa**:
- **Most Seasonal**: Highest amplitude (183.5%)
- **Pattern**: Peak in November, Low in June
- **Variability**: High CV (32.1%), High std (0.210)
- **Interpretation**: Strong seasonal tourism, possibly driven by climate or cultural events

**South America**:
- **Highly Seasonal**: Second highest amplitude (165.5%)
- **Pattern**: Peak in August, Low in April
- **Variability**: Highest CV (35.8%), High std (0.176)
- **Interpretation**: Strong summer peak, significant off-season decline

**North America**:
- **Moderately Seasonal**: Third highest amplitude (116.8%)
- **Pattern**: Peak in August, Low in February
- **Variability**: Moderate CV (24.8%), Moderate std (0.167)
- **Interpretation**: Clear summer peak, winter low

**Asia**:
- **Moderately Seasonal**: Moderate amplitude (76.9%)
- **Pattern**: Peak in August, Low in June
- **Variability**: Moderate CV (24.8%), Lower std (0.153)
- **Interpretation**: Summer peak, relatively stable otherwise

**Europe**:
- **Less Seasonal**: Lower amplitude (55.6%)
- **Pattern**: Peak in August, Low in December
- **Variability**: Lower CV (25.2%), Lower std (0.154)
- **Interpretation**: Summer peak, but maintains higher baseline tourism

**Oceania**:
- **Least Seasonal**: Lowest amplitude (51.0%)
- **Pattern**: Peak in November, Low in May
- **Variability**: Lowest CV (15.7%), Lowest std (0.106)
- **Interpretation**: Most stable tourism levels, minimal seasonal variation

---

## Conclusion

### Answer to Research Question

**Yes, certain regions are significantly more affected by seasonal tourism trends than others.**

### Key Conclusions

1. **Most Affected Regions**:
   - **Africa** and **South America** are most affected by seasonal trends
   - Both show 150-180% increase from low to peak season
   - High variability (CV > 30%) indicates strong seasonal patterns

2. **Least Affected Regions**:
   - **Oceania** is least affected (CV = 15.7%, Amplitude = 51.0%)
   - **Europe** also shows relatively stable patterns (CV = 25.2%, Amplitude = 55.6%)

3. **Regional Patterns**:
   - Most regions peak in **August** (summer), but timing varies
   - Low seasons are more diverse, ranging from February to December
   - Southern Hemisphere regions (Africa, Oceania, South America) show different patterns than Northern Hemisphere

4. **Implications**:
   - Tourism-dependent economies in highly seasonal regions (Africa, South America) may experience significant economic fluctuations
   - Regions with stable patterns (Oceania, Europe) may have more predictable tourism revenue
   - Understanding these patterns can inform:
     - Marketing strategies (targeting peak seasons)
     - Infrastructure planning (capacity for peak seasons)
     - Economic planning (managing off-season impacts)

---

## Technical Summary

### Preprocessing Outputs
- **11 CSV files** saved to `Question3/preprocessed_data/`
- **6 main datasets** for analysis
- **5 key metrics** calculated per continent

### Analysis Outputs
- **6 visualizations** created
- **Statistical summaries** for all continents
- **Rankings** by multiple metrics

### Data Quality
- **0 missing values**
- **1,456 complete records**
- **14 cities** across **6 continents**
- **104 weeks** of data (Dec 2023 - Dec 2025)

---

## Files and Datasets

### Preprocessing Notebook
- **File**: `Question3/q3preprocessing.ipynb`
- **Outputs**: 11 CSV files in `Question3/preprocessed_data/`

### EDA Notebook
- **File**: `Question3/q3EDA.ipynb`
- **Inputs**: Preprocessed CSV files
- **Outputs**: 6 visualizations, statistical summaries

### Documentation
- **File**: `Question3/Question3doc.md` (this document)

---

*Documentation created by Senior Data Analyst*
*Analysis Date: 2024*
*Project: Data Analysis - Seasonal Tourism Patterns*

