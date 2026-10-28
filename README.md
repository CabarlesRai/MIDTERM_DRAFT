
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
![Preprocessing Flowchart](images/VCT%20Paris%202025_Flowchart_Bautista_Cabarles_page-0001.jpg)

- **Decisions in our flowchart:**
  
| Decision | What we chose and why |
|----------|-----------------------|
| **Missing values?** | **Yes:** we filled `map_name` and `map_winner` with `'All maps'` (340 NaN each, same rows) and kept every row. The NaNs appear only in `stat_type = 'overall'` rows, because an overall row covers every map, so there is no single map to name. |
| **Duplicate rows?** | **No:** `df.duplicated().sum()` found 0 duplicate rows, so nothing was dropped and the data was kept as is. |
| **Outliers? (IQR 1.5×)** | **Yes:** we winsorized `rating`, `acs`, `adr`, `k`, `d`, `a`, `fk`, `fd` by clipping them to Q1 − 1.5·IQR and Q3 + 1.5·IQR. The values are real player performances, so we capped them instead of deleting rows. |
| **i < 10 ?** | **Yes** for charts 1 to 9: `i = i + 1` and we draw the next chart. **No** once `i` reaches 10: all 10 charts are drawn, so we go to END. |

- **Changes we made:**
  
The final code differs from our first flowchart in the following ways.

| Initial Flowchart | Final Flowchart | Why we changed it |
|---|---|---|
| The code one-hot encodes `stat_type`, `map_name`, `map_winner`, `event_stage`, `player_team`, `team1` and `team2`, multi-hot encodes `agent`, and drops `event_name`. | The code only drops `event_name` and one-hot encodes `stat_type`. | The charts and PCA use numeric stats, so the extra map, team, stage and agent encodings would only add many sparse columns. |
| The code saves each chart as `figures/viz_{i}.png`. | The code no longer saves the charts as PNG files. | Charts display inline in Colab, so saving PNG files was not needed. |
| There's no code to keep a raw copy of the data. | Added code to keep a raw copy of the data: `df_raw = df.copy()`. | Needed to compare raw data against cleaned data in the Data Quality Report and the missingness chart (chart 6). |
| There's no code to keep a copy of the data before winsorizing. | Added `df_before_outliers = df.copy()` right before winsorizing. | Chart 8 (boxplot) compares the data before and after outlier handling. |
| The code winsorizes the outliers and then moves straight on to encoding. | Added a snapshot, `df_clean = df.copy()`, taken before encoding and scaling. | The charts need a readable version of the data before encoding or scaling. |
| There's no data quality report. | Added a Data Quality Report that compares missing values in `df_raw` and `df_final`, expecting 0 after cleaning. | Added to show that the cleaning removed all missing values. |
| There's no code to prepare a separate dataset for the charts. | Added code that takes the overall rows of `df_clean` and creates a Won/Lost `result` column. | Gives every chart one consistent subset and a Won/Lost label. |
| The loop draws 10 charts from `df_clean`, filtered by `stat_type`. | The loop draws 10 named charts, and each one uses `overall`, `df_final`, `df_raw` or `df_before_outliers` as needed. | The chart list was finalized, and each chart uses the dataframe that suits it. |

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
### Plot 1: Histogram + KDE: `rating` by result
<p align="center">
  <img width="696" height="479" alt="kde" src="https://github.com/user-attachments/assets/6eb6b484-f178-47c6-9c59-fc6e42db8092" />
</p>

*   **Description** - Distribution of player rating (overall rows), Won vs Lost
*  **Insight** - Rating is roughly bell-shaped and centered near 1.0 (median 0.97), with most player-matches between 0.85 and 1.14 and a slight right skew.

### Plot 2: KDE: `acs` before vs after scaling
  <p align="center">
  <img width="1084" height="443" alt="2" src="https://github.com/user-attachments/assets/0c58bf5a-5127-4c2c-9b6c-4886228e7081" />
</p>

*   **Description** - Density of Average Combat Score, raw and standardized
*   **Insight** - The two panels have the same shape. Scaling only moves the center to 0 and the spread to 1, so ACS can be compared fairly with the other scaled columns.

### Plot 3: Scatter: kills vs deaths
<p align="center">
  <img width="687" height="556" alt="3" src="https://github.com/user-attachments/assets/4e50b83b-6a41-4fa2-985d-8f5b0ceaef06" />
</p>

*  **Description** - Each dot is a player-match, colored by rating
* **Insight** - Kills and deaths rise together (more rounds means more of both), but the high-rating (bright) points sit below the diagonal, where kills exceed deaths.

### Plot 4: Pairplot: `rating`, `acs`, `kast`, `hs_percent` by result
<p align="center">
  <img width="685" height="617" alt="4" src="https://github.com/user-attachments/assets/cf361395-4b0b-4f27-8b11-55229de71e5d" />
</p>

*  **Description** - Pairwise relationships, colored Won vs Lost
* **Insight** - `adr` was left out because it is almost the same as `acs` (r ≈ 0.97). `rating` follows `acs` closely, and `kast` is a looser relationship

### Plot 5: Correlation heatmap (lower triangle)
<p align="center">
  <img width="856" height="755" alt="5" src="https://github.com/user-attachments/assets/17d2e8c7-35c3-4845-9973-f8bc035edc7f" />
</p>

*  **Description** - Pearson correlation of the numeric stats
* **Insight** - `rating` is most tied to `kd_diff` (0.90), `acs` (0.85) and `adr` (0.83). Assists barely matter (0.03), and deaths pull rating down (-0.21).

### Plot 6: Missingness plot: before vs after
<p align="center">
  <img width="1384" height="492" alt="6" src="https://github.com/user-attachments/assets/afabd4c4-f264-4140-90d0-240d2e5893ce" />
</p>
  
*  **Description** - Missing cells in `df_raw` and in `df_final`
* **Insight** - Before: only `map_name` and `map_winner` have gaps (340 rows each), and they are the same rows, the overall stat rows. After: the second panel has no missing cells, so filling with 'All maps' removed every gap without dropping any row.

### Plot 7: Countplot: records per `event_stage`
<p align="center">
  <img width="1053" height="479" alt="7" src="https://github.com/user-attachments/assets/59b65768-d6b3-4e7b-8ad2-5b319389cb5f" />
</p>

*  **Description** - Player-match records per tournament stage (overall rows)
* **Insight** - Each stage now appears as one clean label (the newline/tab junk was removed in Step 6). Stages with more matches have more records.

### Plot 8: Boxplot: `acs` and `k` before vs after winsorizing
<p align="center">
  <img width="983" height="492" alt="8" src="https://github.com/user-attachments/assets/405ca912-1acc-4bd8-a390-35a9aab49766" />
</p>

*  **Description** - Spread and outliers, before and after the clip
* **Insight** - The extreme points above the whisker in the Before box are pulled to the upper limit in the After box, while the box and median barely move. Winsorizing caps the extremes without changing the typical player.

### Plot 9: Line plot: matches per date
<p align="center">
  <img width="1011" height="537" alt="9" src="https://github.com/user-attachments/assets/e11c2821-3ae1-4c53-91e3-a9cc0115d1d1" />
</p>

*  **Description** - Number of matches played on each date
* **Insight** - Shows the tournament schedule: busy days and rest days.
  
### Plot 10: PCA scatter by result
<p align="center">
  <img width="699" height="556" alt="10" src="https://github.com/user-attachments/assets/a98e270b-1826-44cb-8f06-419ee15c0e99" />
</p>

*  **Description** - The 12 scaled stats projected to 2 components, colored Won vs Lost	
* **Insight** - PC1 captures the most variance (the percentage is in the axis label).

## 💡Key Insights
1. **Performance is centered at average.** Player rating averages about 0.99, with most players between 0.85 and 1.14.
2. **Rating is driven by damage and kill balance**, not assists. `kd_diff`, `acs` and `adr` show the strongest links with rating.
3. **The only missing data was structural.** `map_name` and `map_winner` were empty exactly where the stats cover all maps combined, so nothing was lost by filling them.
4. **Winners and losers differ in rating.** Charts 1, 4 and 10 compare players by match result.
5. **Preprocessing changed the scale, not the story.** Chart 2 shows scaling keeps the shape of ACS, and chart 8 shows winsorizing only caps the extreme tail.
## 🚀 How to Run
- Open the notebook in Google Colab.
- Install dependencies from `requirements.txt`.
