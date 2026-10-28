
## 👥 Group Information
- **Group Name:** Group 3
- **Members:** Bautista, Dean Mark | Cabarles, Raiza
- **Assigned Theme:** Sport
- **Topic:** Valorant E-sports

## 📊 Project Overview
- **Dataset Title:** Valorant Champions Tour Paris 2025 
- **Source (URL or reference):** https://www.kaggle.com/datasets/piyush86kumar/valorant-champions-tour-2025-paris
- **License/Attribution:** Attribution 4.0 International (CC BY 4.0)
- **Objective:** This dataset analyzes player performance, agent selection, and team statistics across all regions in the VCT 2025 Paris events. It aims to identify top-performing players, effective agents, and key performance indicators, such as ACS, K/D difference, and first kills, that may influence match outcomes.

## 🔀 Flowchart
- Embed your flowchart image here:
  ```markdown
  ![Preprocessing Flowchart](Flowchart/Valorant_Champion_2025-Paris_Bautista_Cabarles_Flowchart.png)
  ```
- **Decisions in our flowchart:** list each decision diamond and what you chose on each branch.
- **Changes we made:** if the code ended up different from your first flowchart, say what changed and why.

## 🧹Preprocessing Summary
The steps were applied in order to produce the cleaned dataset. 
1.  Initialize the dataset by importing necessary libraries and creating an output folder.
2.  Load the CSV file.
1.   Inspect the data shape (1,220 rows × 26 columns), types, and summary statistics.
2.   Keep a raw copy (used later for quality report ansd chart 6).
1.   Check for missing values. There are 680 NaNs (340 each in `map_name` and `map_winner`), all in stat_type = 'overall' rows, so every row is kept. They are filled with `All maps` instead of dropping rows or imputing a median or mode.
2.   Check for Duplicate Rows. If any exist, `drop_duplicates()`, keep the first copy and reset the index.
1.   Fix data types:
    - `match_date` → converted to datetime
    - `kast`, `hs_percent` → `%` stripped and converted to float
    - `event_stage` → newline/tab junk stripped
    - `score_overall` (e.g. "2 - 0") → split into 2 integer columns
2.   Keep a copy before winsorizing (`df_before_outliers = df.copy()`), used by chart 8.
1.   Outliers: Outliers in rating, acs, adr, k, d, a, fk, and fd are detected with the `IQR 1.5× rule` and winsorized, clipped to `Q1 − 1.5·IQR` and `Q3 + 1.5·IQR`, so no rows are removed.
2.   Take a snapshot `df_clean = df.copy()` before encoding and scaling, used by the charts.
1.   Encode categorical columns:
      - Create `df_final` from `df_clean` without `event_name` (constant)
      - One-hot encode `stat_type` only
      - Keep the other text columns as labels

1.   Scaling: `rating`, `acs`, `k`, `d`, `a`, `kd_diff`, `kast`, `adr`, `hs_percent`, `fk`, `fd` was scaled using Standardscaler.
2.   Save:  Export the cleaned dataset to `data/vct_paris_cleaned.csv`
1.   Run a data quality report. Compare missing values before `df_raw` and after `df_final` ; the expected result is 0.

## 📈 Visualizations

- Total: **10**
- Short description + insight for each plot.

## 💡Key Insights
- 3 to 5 main findings from your data.

## 🚀 How to Run
- Open the notebook in Google Colab.
- Install dependencies from `requirements.txt`.
