
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

| Decision | What we chose and why |
|----------|-----------------------|
| **Missing values?** | **Yes:** we filled `map_name` and `map_winner` with `'All maps'` (340 NaN each, same rows) and kept every row. The NaNs appear only in `stat_type = 'overall'` rows, because an overall row covers every map, so there is no single map to name. |
| **Duplicate rows?** | **No:** `df.duplicated().sum()` found 0 duplicate rows, so nothing was dropped and the data was kept as is. |
| **Outliers? (IQR 1.5×)** | **Yes:** we winsorized `rating`, `acs`, `adr`, `k`, `d`, `a`, `fk`, `fd` by clipping them to Q1 − 1.5·IQR and Q3 + 1.5·IQR. The values are real player performances, so we capped them instead of deleting rows. |
| **i < 10 ?** | **Yes** for charts 1 to 9: `i = i + 1` and we draw the next chart. **No** once `i` reaches 10: all 10 charts are drawn, so we go to END. |

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
**Plot 1 : Histogram + KDE: `rating` by result**


*   **Description** - Distribution of player rating (overall rows), Won vs Lost
*  **Insight** - Rating is roughly bell-shaped and centered near 1.0 (median 0.97), with most player-matches between 0.85 and 1.14 and a slight right skew.

**Plot 2 : KDE: `acs` before vs after scaling**


*   **Description** - Density of Average Combat Score, raw and standardized
*   **Insight** - The two panels have the same shape. Scaling only moves the center to 0 and the spread to 1, so ACS can be compared fairly with the other scaled columns.

**Plot 3: Scatter: kills vs deaths**
*  **Description** - Each dot is a player-match, colored by rating
* **Insight** - Kills and deaths rise together (more rounds means more of both), but the high-rating (bright) points sit below the diagonal, where kills exceed deaths.

**Plot 4: Pairplot: `rating`, `acs`, `kast`, `hs_percent` by result**
*  **Description** - Pairwise relationships, colored Won vs Lost
* **Insight** - `adr` was left out because it is almost the same as `acs` (r ≈ 0.97). `rating` follows `acs` closely, and `kast` is a looser relationship

**Plot 5: Correlation heatmap (lower triangle)**
*  **Description** - Pearson correlation of the numeric stats
* **Insight** - `rating` is most tied to `kd_diff` (0.90), `acs` (0.85) and `adr` (0.83). Assists barely matter (0.03), and deaths pull rating down (-0.21).

**Plot 6: Missingness plot: before vs after**
*  **Description** - Missing cells in `df_raw` and in `df_final`
* **Insight** - Before: only `map_name` and `map_winner` have gaps (340 rows each), and they are the same rows, the overall stat rows. After: the second panel has no missing cells, so filling with 'All maps' removed every gap without dropping any row.

**Plot 7: Countplot: records per `event_stage`**
*  **Description** - Player-match records per tournament stage (overall rows)
* **Insight** - Each stage now appears as one clean label (the newline/tab junk was removed in Step 6). Stages with more matches have more records.

**Plot 8: Boxplot: `acs` and `k` before vs after winsorizing**
*  **Description** - Spread and outliers, before and after the clip
* **Insight** - The extreme points above the whisker in the Before box are pulled to the upper limit in the After box, while the box and median barely move. Winsorizing caps the extremes without changing the typical player.

**Plot 9: Line plot: matches per date**
*  **Description** - Number of matches played on each date
* **Insight** - Shows the tournament schedule: busy days and rest days.
  
**Plot 10: PCA scatter by result**
*  **Description** - The 12 scaled stats projected to 2 components, colored Won vs Lost	
* **Insight** - PC1 captures the most variance (the percentage is in the axis label).


## 💡Key Insights
- 3 to 5 main findings from your data.

## 🚀 How to Run
- Open the notebook in Google Colab.
- Install dependencies from `requirements.txt`.
