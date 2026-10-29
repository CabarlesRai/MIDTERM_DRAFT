
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

![Flowchart 1: Data Preprocessing](images/final%20FLOWCHART%201.drawio_page-0001.jpg)

---

![Flowchart 2: Data Visualization](images/final%20Flowchart_2_Visualization.drawio_page-0001.jpg)

---

> ### Decisions Summary
  
Flowchart 1 (Data Preprocessing) has five decision diamonds. Flowchart 2 (Data Visualization) has none: it runs from START through the ten charts to END.

| # | Decision (condition) | What we chose | Justification |
|---|---|---|---|
| 1 | **Are missing values present?** | **Yes**: fill `map_name` and `map_winner` with `'All maps'`, keep every row, then confirm 0 missing values are left. (If **No**: move on to the duplicate check.) | The 340 missing values per column appear only in `stat_type = 'overall'` rows. An overall row covers every map, so there is no single map to name, which means the gaps are structural and not data errors. Filling them with `'All maps'` keeps all rows and keeps the meaning clear. A copy of the raw data (`df_raw`) is kept for the missingness chart. |
| 2 | **Are there duplicate rows?**  | **No**: the check found 0 duplicate rows, so we display "Duplicate rows found: 0 / No duplicate rows, nothing to drop." and move on to fixing the data types. | Every row is a unique player-match (or player-map) record, so there is nothing to remove and all rows are kept. The check stays in the pipeline because duplicated rows would double-count a player's stats and bias every average, correlation and chart. |
| 3 | **Are outliers present?**  | **Yes**: **winsorize**, clipping each value to `Q1 − 1.5·IQR` and `Q3 + 1.5·IQR`, with bounds computed separately for each `stat_type`. All rows are kept.| The extreme values are real player performances, so deleting the rows would throw away genuine data. Capping limits their pull on means, correlations and scaling. Bounds are separate for `overall` and `map` rows because totals like `k`, `d` and `a` are naturally larger for an overall row (all maps combined); one shared bound would wrongly flag those rows. A pre-winsorizing copy (`df_before_outliers`) lets chart 8 still show the real star performances. |
| 4 | **Are the variables categorical?**  | **Yes**: drop `event_name` and one-hot encode **`stat_type` only** (`stat_type_map`, `stat_type_overall`). | `stat_type` changes what every row *means* (all maps vs. a single map) and has only 2 values, so it adds just 2 columns. `event_name` has a single value, so it carries no information. The remaining columns (team, agent, map, stage) stay as readable text labels so charts, legends and tables show real names, and encoding them would add many columns without improving the results. |
| 5 | **Are the variables numerical?**  | **Yes**: apply **StandardScaler** (mean 0, std 1) to the 12 performance columns (`rating, acs, k, d, a, kd_diff, kast, adr, hs_percent, fk, fd, fk_fd_diff`), then check that mean ≈ 0 and std ≈ 1. | The metrics sit on very different scales (e.g. `acs` in the hundreds vs. `rating` near 1), so standardizing puts them on equal footing for comparison. `match_id`, `player_id` and the one-hot columns are **not** scaled, because they are identifiers or 0/1 flags. Most charts use `df_clean` (taken before scaling) so the axes stay in real units. |

----

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
2.  Upload and Load the CSV file.
3.   Inspect the data shape , types, and summary statistics.
4.   Keep a raw copy (used later for quality report ansd chart 6).
5.   Check for missing values. There are 680 NaNs (340 each in `map_name` and `map_winner`), all in stat_type = 'overall' rows, so every row is kept. They are filled with `All maps` instead of dropping rows or imputing a median or mode.
6.   Check for Duplicate Rows. If any exist, `drop_duplicates()`, keep the first copy and reset the index.
7. Fix data types:
    - `match_date` → converted to datetime
    - `kast`, `hs_percent` → `%` stripped and converted to float
    - `event_stage` → newline/tab junk removed
    - `score_overall` (e.g. "2 - 0") → split into two integer columns

8. Display the data types and the first rows after fixing
9.   Select outlier columns: `rating`, `acs`, `adr`, `k`, `d`, `a`, `fk`, `fd`. Keep a copy as `df_before_outliers`.
10.  IQR: Compute `Q1`, `Q3`, `IQR`, and the bounds `Q1 − 1.5·IQR, Q3 + 1.5·IQR` per column, separately for each stat_type (overall vs. map). Count the outliers and display the results.
11. Outliers Present?:
   - `Yes` → winsorize by clipping each value to its bound.. These are real player stats, so every row is kept. Then display a confirmation message with `len(df)`
   - `No`  → skip this step
12.   Take a snapshot `df_clean = df.copy()` before encoding and scaling, used by the charts.
13.   Encode categorical columns:
      - Drop the constant `event_name`
      - One-hot encode `stat_type` only
      - Keep the other text columns as labels
      - Display the encoded columns
14.   Scaling: `rating`, `acs`, `k`, `d`, `a`, `kd_diff`, `kast`, `adr`, `hs_percent`, `fk`, `fd`and `fk_fd_diff` was scaled using Standardscaler. Display the scaled dataset.
15.   Save:  Export the cleaned dataset to `data/vct_paris_cleaned.csv`
16.   Verify: Display the shape of the saved file and should match `df_final`
17.   Run a data quality report. Compare missing values before `df_raw` and after `df_final` ; the expected result is 0.



## 📈 Visualizations
### **Plot 1 : Histogram - Distribution of ACS by Playoff Team**
<p align="center">
  <img width="1222" height="694" alt="1  kde" src="https://github.com/user-attachments/assets/6e553812-a396-4223-860b-6b92dbfbb344" />
</p>

*   **Description** - The distribution of Average Combat Score `(ACS)` for the eight playoff teams. The x-axis is `ACS` (about 50 to 330), and the y-axis is density.
*  **Insight** - The histogram shows that most playoff teams have an `Average Combat Score (ACS)` between 130 and 230, with the highest density around 150–220. `MIBR` and `G2 Esports` show noticeable peaks in this range, while a few teams have higher ACS values above 300, indicating differences in individual combat performance.

### **Plot 2 : KDE: Density Distribution of Player Ratings by Map**
<p align="center">
  <img width="1112" height="650" alt="Density Distribution of Player Ratings by Map" src="https://github.com/user-attachments/assets/cc6ec917-6cb9-404b-869f-0183165e60f6" />
</p>

*   **Description** - Smoothed KDE curves of player rating for seven maps, with rating on the x-axis (about 0 to 2.2) and density on the y-axis.
*   **Insight** - Most of the maps shows player ratings concentrated between 1.00 and 1.25 that indicates manny players perform at this range. `Bind` has the sharpest peak above 1.50, while `Haven` has flatter distribution. 

### **Plot 3: Scatter Plot — Relationship Between Kill-Death Difference and Rating by Outcome**
<p align="center">
  <img width="1002" height="672" alt="Scatter Plot: Kill-Death Difference vs Rating by Outcome" src="https://github.com/user-attachments/assets/c2a5fdae-3119-44ff-b57d-47d34b53215b" />
</p>

*  **Description** - Each point is a player-map performance, colored by whether that player's team won or lost the map.
* **Insight** - The plot shows a strong positive correlation between kill-death difference and player rating. Players with higher kill-death difference tend to achieve higher rating and most winning performances.


### **Plot 4: Bar Plot — Agent Pick Frequency Across All Maps**
<p align="center">
  <img width="1497" height="672" alt="Bar Plot: Agent Pick Frequency Across All Maps" src="https://github.com/user-attachments/assets/a5ba4191-d9bc-47c9-9859-218728708427" />
</p>

*  **Description** - Chart shows how often each agent was picked across all maps. The x-axis lists 23 agents, the y-axis shows pick count and the five most-picked agents are highlighted.
* **Insight** - The most frequent picked agent is `Omen(141)` appearances, followed by `Sova(94)` and `Yoru(93)`. Meanwhile, `Skye` has the fewest picks with only 3 appearances. This suggest that certain agents are favorable among player. 

### **Plot 5: Pair Plot — Survival and Support Performance by Top 5 Agent**
<p align="center">
  <img width="995" height="911" alt="Pair Plot of Player Performance Metrics" src="https://github.com/user-attachments/assets/507ad2af-7a68-419e-849b-3947b90c2283" />
</p>

*  **Description** - The plot uses deaths, assists, KAST, and ADR for the five most frequently picked agents.
* **Insight** - `KAST` and `ADR` have positive correlation which means that players who contribute effectively tend to deal more damage. While, `Assist` and `Death` suggest weak relationship.

### **Plot 6: Correlation Heatmap — Correlation Between Overall Combat Performance Metrics**
<p align="center">
  <img width="1023" height="892" alt="Heatmap of Player Performance Metrics" src="https://github.com/user-attachments/assets/01609455-8537-4576-a0a2-6fb89228d382" />
</p>

*  **Description** - Pearson correlations are calculated from overall player rows. Only the lower triangle is shown, so each pair appears once, and derived difference metrics are excluded to avoid redundant comparisons.
* **Insight** -The heatmap shows a strong positive correlation between `ACS` and `ADR` with `r=0.97`. `Rating` is also strongly correlated with `ACS` `(r=0.85)` and  `ADR` `(0.83)`. In contrast, `HS%` has weak relationship with other metrics.

### **Plot 7: Missingness Plot — Before vs After**
<p align="center">
  <img width="1772" height="848" alt="Missingness Plot of the Dataset Before Cleaning" src="https://github.com/user-attachments/assets/d8be2e27-dbbe-42b1-8e6c-0c4ad2c4d4fb" />
</p>

*  **Description** - Each column of the dataset is one vertical strip and each row is one record; black cells are missing values. The left panel is the raw data `(df_raw)` and the right panel is the cleaned data `(df_final)`. Columns with gaps are labelled with their % missing. Column names are rotated 90° and centred under their own strip so both panels line up.
* **Insight** - In initial dataset, there were 680 missing cells, mainly in the `map_name` and `map_winner` columns. After data cleaning, all missing values were handled with 0 missing cells.

### **Plot 8: Violin Plot — Player Rating Distribution by Top 5 Agent**
<p align="center">
  <img width="1112" height="672" alt="Violin Plot of Player Performance Distribution" src="https://github.com/user-attachments/assets/509f7f41-275a-4fa7-868d-01a4498c1b01" />
</p>

*  **Description** - Each violin shows the full shape of an agent's rating distribution, with the box inside marking the median and the middle 50%. Agents are ordered from the highest to the lowest median rating. This uses the pre-winsorized data, so star performances stay visible in the tails.
* **Insight** - It shows that player ratings are concentrated around 1.0, while some extend toward 1.5 and above. `Yoru` has wider distribution while `Sova`has the highest peak.

### **Plot 9: Temporal Line Plot — Team Average Rating by Event Stage**
<p align="center">
  <img width="1277" height="694" alt="Temporal Line Plot of Player Performance Over Time" src="https://github.com/user-attachments/assets/c0c46029-7586-4f5e-9d4c-c152d4dc8956" />
</p>

*  **Description** - The x-axis follows the tournament stage order determined by each stage's first match date. The eight most active teams are shown to avoid a crowded legend.
* **Insight** - EThe line plot shows that team average rating varies across event stages. `NRG` and `MIBR` reach the highest rating between 1.2-1.3. `GIANTX` experinces a notable drop during the playoffs.

### **Plot 10: Team Heatmap — Team Performance Profiles**
<p align="center">
  <img width="946" height="846" alt="Team Heatmap of Playoff Team Performance" src="https://github.com/user-attachments/assets/fdae0edc-a476-45d8-842a-d468d28448e3" />
</p>

*  **Description** - Each cell shows a team's mean metric standardized across teams (z-score within each metric). This puts `rating`, `ACS`, `ADR`, and `KAST` on a comparable color scale.
* **Insight** - The heatmap shows `MIBR` has the highest strongest overall performance having the highest standardized values in `Rating`, `ACS` and `KAST`. Followed by `NRG` and `Paper Rex`. In contrast, `Bilibili Gaming` and `Rex Regumm Qeon` have the lowest values. 

## 💡Key Insights
1. **Performance is centered at average.** Player rating averages about 0.99, with most players between 0.85 and 1.14.
2. **Rating is driven by damage and kill balance**, not assists. `kd_diff`, `acs` and `adr` show the strongest links with rating, while `assists` are almost unrelated.
3. **The only missing data was structural.** `map_name` and `map_winner` were empty exactly where the stats cover all maps combined, so nothing was lost by filling them.
4. **Winners and losers differ in rating.** Chart 3 compares players by match result: below about -10 `kd_diff` almost every point is a lost map, and above about 10 almost every point is a win.
5. **Preprocessing changed the scale, not the story.** Chart 8 uses the pre-winsorized data, so star performances stay visible (Sova's tail reaches higher than 2.5), and winsorizing only capped the extreme tail.

## 🚀 How to Run
- Open the notebook in Google Colab.
- Install dependencies from `requirements.txt`.
