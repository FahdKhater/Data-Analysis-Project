# Global Travel Analysis: Airbnb Prices vs. Satisfaction 

## Project Overview
This project investigates the relationship between the cost of accommodation (Airbnb average prices) and the overall travel satisfaction score of major global cities. The goal was to determine if paying a premium for accommodation correlates with a better travel experience, or if budget-friendly destinations offer comparable satisfaction.

## 📊 Key Findings
* **Correlation:** There is a **moderate positive correlation** ($r \approx 0.48$) between Airbnb prices and travel satisfaction.
* **Statistical Significance:** With a p-value of **0.079**, the result is **not statistically significant** at the standard 0.05 confidence level.
* **Conclusion:** While the data suggests a trend where expensive cities generally have higher satisfaction scores, we cannot rule out random chance, likely due to the limited sample size ($n=14$ cities).
* **Outliers:** **Amsterdam** represents a high-cost/high-satisfaction outlier, while **Tokyo** presents exceptional value (high satisfaction with moderate pricing).

## 📂 Dataset
The analysis is based on two datasets:
1.  **`airbnb_scraped_dataset.csv`**: Contains raw listing data (Price, City, Room Type) for various international cities.
2.  **`Travel_Satisfaction_Index.csv`**: Contains the `Travel_Score` (Satisfaction Index) for corresponding cities.

## 🛠️ Technologies Used
* **Python 3**
* **Pandas** (Data Cleaning & Merging)
* **Seaborn & Matplotlib** (Visualization)
* **SciPy** (Statistical Analysis - Pearson Coefficient)

## Visualizations
The project generates three key visualizations to interpret the data:

1.  **Labeled Scatter Plot:**
    * *Purpose:* Identifies the correlation and specific city outliers.
    * *Insight:* Shows the spread of cities around the regression line.

2.  **Dual-Axis Chart:**
    * *Purpose:* Decouples price and satisfaction to highlight gaps.
    * *Insight:* Easily identifies "Value" cities (High Satisfaction bar, Low Price line) vs. "Overpriced" cities.

3.  **Price Distribution Boxplot:**
    * *Purpose:* Shows the variance of Airbnb prices within each city.
    * *Insight:* Reveals if a city's high average price is driven by luxury outliers or a consistently high cost of living.

## How to Run
1.  Ensure you have the required libraries installed:
    ```bash
    pip install pandas matplotlib seaborn scipy numpy
    ```
2.  Place the CSV datasets in the same directory as the notebook.
3.  Run the Jupyter Notebook `DA_Project_Q1.ipynb`.