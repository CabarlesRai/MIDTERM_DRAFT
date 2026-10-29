
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

![Preprocessing Flowchart](images/VCT%20Paris%202025_Flowchart_Bautista_Cabarles_page-0001.jpg)

> ### Decisions Summary
  
Flowchart 1 (Data Preprocessing) has five decision diamonds. Flowchart 2 (Data Visualization) has none: it runs from START through the ten charts to END.

| # | Decision (condition) | What we chose | Justification |
|---|---|---|---|
| 1 | **Are missing values present?** | **Yes**: fill `map_name` and `map_winner` with `'All maps'`, keep every row, then confirm 0 missing values are left. (If **No**: move on to the duplicate check.) | The 340 missing values per column appear only in `stat_type = 'overall'` rows. An overall row covers every map, so there is no single map to name, which means the gaps are structural and not data errors. Filling them with `'All maps'` keeps all rows and keeps the meaning clear. A copy of the raw data (`df_raw`) is kept for the missingness chart. |
| 2 | **Are there duplicate rows?**  | **No**: the check found 0 duplicate rows, so we display "Duplicate rows found: 0 / No duplicate rows, nothing to drop." and move on to fixing the data types. | Every row is a unique player-match (or player-map) record, so there is nothing to remove and all rows are kept. The check stays in the pipeline because duplicated rows would double-count a player's stats and bias every average, correlation and chart. |
| 3 | **Are outliers present?**  | **Yes**: **winsorize**, clipping each value to `Q1 − 1.5·IQR` and `Q3 + 1.5·IQR`, with bounds computed separately for each `stat_type`. All rows are kept.| The extreme values are real player performances, so deleting the rows would throw away genuine data. Capping limits their pull on means, correlations and scaling. Bounds are separate for `overall` and `map` rows because totals like `k`, `d` and `a` are naturally larger for an overall row (all maps combined); one shared bound would wrongly flag those rows. A pre-winsorizing copy (`df_before_outliers`) lets chart 8 still show the real star performances. |
| 4 | **Are the variables categorical?**  | **Yes**: drop `event_name` and one-hot encode **`stat_type` only** (`stat_type_map`, `stat_type_overall`). | `stat_type` changes what every row *means* (all maps vs. a single map) and has only 2 values, so it adds just 2 columns. `event_name` has a single value, so it carries no information. The remaining columns (team, agent, map, stage) stay as readable text labels so charts, legends and tables show real names, and encoding them would add many columns without improving the results. |
| 5 | **Are the variables numerical?**  | **Yes**: apply **StandardScaler** (mean 0, std 1) to the 12 performance columns (`rating, acs, k, d, a, kd_diff, kast, adr, hs_percent, fk, fd, fk_fd_diff`), then check that mean ≈ 0 and std ≈ 1. | The metrics sit on very different scales (e.g. `acs` in the hundreds vs. `rating` near 1), so standardizing puts them on equal footing for comparison. `match_id`, `player_id` and the one-hot columns are **not** scaled, because they are identifiers or 0/1 flags. Most charts use `df_clean` (taken before scaling) so the axes stay in real units. |

> ### Changes Summary
  
The final code differs from our first flowchart in the following ways.

| Initial flowchart | Final flowchart | Why we changed it |
|---|---|---|
| **One combined flowchart** covering preprocessing and visualization. | **Two flowcharts**: Flowchart 1 (Data Preprocessing) and Flowchart 2 (Data Visualization). | Preprocessing and visualization are two separate stages. Splitting them keeps each one readable and lets Flowchart 2 show every chart in detail. |
| Steps have **no display boxes**, only one *Inspect the data* box. | Almost every step is **followed by a display box**: inspect output, missing values left, duplicate count, data types after fixing, outlier table, encoded columns, scaled-data check, saved file shape, quality report totals. | Each display box matches something the notebook actually prints, so the flowchart and code can be compared line by line. |
| **Outliers:** keep a copy, ask "Outliers? (IQR 1.5×)", then winsorize. | **Select the columns and keep the copy**, **calculate Q1, Q3, IQR and bounds separately for each `stat_type`**, **display** the bounds and counts, then winsorize and display the message. | Totals such as `k`, `d`, `a`, `fk` and `fd` are naturally larger in `overall` rows (all maps combined). One shared bound would flag them as outliers only because they cover more maps. |
| **Prepare chart data:** `overall` rows of `df_clean` and a Won / Lost result per player-match. | `overall` **and map rows** of `df_clean`, a Won / Lost result, **short team names, `PLAYOFF_TEAMS`, theme and team colours**, and creates the `figures/` folder. | Map and agent charts need map-level rows, because overall rows list several agents per player (e.g. `Yoru, Sova`). The shared theme and fixed team colours keep every chart consistent. |
| **Loop:** `i = 1`, `total = 10`, "Draw chart i", check `i < 10?`. | **No loop.** Ten charts run in sequence, each with its own *generate* box and its own *display* box. | Each chart has different variables and its own display text. Separate boxes show that clearly, and the notebook now has one code cell and one displayed chart per chart. |
| Final step: **Save the cleaned dataset** only. | Adds **Save each chart to the figures folder** (Colab: download the cleaned CSV and `figures.zip`). | Every chart is saved as a PNG, and the cleaned data and figures can be downloaded together from Colab. |

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
