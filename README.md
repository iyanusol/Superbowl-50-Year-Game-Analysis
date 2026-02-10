# Superbowl 50 Year Game Analysis (Power BI)
An analytical overview of 50 years of Super Bowl games, halftime performances, ads, and viewership trends built in Power BI.
![Game Overview Main](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/Game%20Overview%20Dashboard%20Main.png)


## Project Overview

This project consolidates fragmented Super Bowl data spanning five decades into a single analytical model. It demonstrates a full data analytics workflow from data sourcing and cleaning to modeling, DAX calculations, and interactive visualization.

The dashboards enable year-by-year exploration of:
- Game outcomes and scoring margins
- Team dominance and losses
- MVPs, coaches, and quarterbacks
- Viewership vs stadium attendance
- Halftime artist performance
- Advertising category presence


![Game Overview Drill](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/Game%20Overview%20Dashboard%20Drill.png)



## Data Sources (Reliable)
This project combines multiple trusted sources into one dataset:
- NFL historical Super Bowl game results (teams, score, MVP, venue, city/state)
- Broadcast/network and audience metrics (viewership, household share, ratings)
- Ads and commercial metadata (ad cost and product categories)
- Halftime performance info (artists, number of songs, performance ratings)

Note: Where only recent years had specific fields (example: ad cost, certain viewership metrics, or additional attributes), I extended the dataset by sourcing missing historical values from reliable public references and validating consistency across sources before merging.

## Workflow Summary (End-to-End)
### Step 1: Data Collection
- Gathered the primary Super Bowl dataset (game, location, winners/losers, points, MVP)
- Gathered supporting datasets:
  - halftime musician data (artists, performance rating, songs)
  - viewership/ratings + attendance
  - ads by product type + ad cost metrics

### Step 2: Python Data Cleaning and Standardization
Python was used to:
- standardize text formatting (team/artist names, trimming spaces, casing)
- fix missing values and invalid numeric types
- merge multiple year files into a single consolidated dataset
- export clean CSVs for Excel + Power BI import

#### Python Code Example 1: Cleaning + Standardizing Fields
```python
import pandas as pd

df = pd.read_csv("raw_superbowl_data.csv")

# standardize text columns
text_cols = ["winner", "loser", "mvp", "stadium", "city", "state"]
for c in text_cols:
    df[c] = (
        df[c]
        .astype(str)
        .str.strip()
        .str.replace(r"\s+", " ", regex=True)
    )

# numeric cleanup
df["winner_pts"] = pd.to_numeric(df["winner_pts"], errors="coerce")
df["loser_pts"] = pd.to_numeric(df["loser_pts"], errors="coerce")
df["attendance"] = pd.to_numeric(df["attendance"], errors="coerce")

# fill missing numeric values (example strategy)
df["attendance"] = df["attendance"].fillna(df["attendance"].median())

df.to_csv("superbowl_cleaned.csv", index=False)
```
### Step 3: Excel Consolidation and Gap Filling

Excel was used to:
- append additional year blocks to build a full 50-year timeline
- validate totals and remove duplicates
- enforce consistent date formatting and text standards
- fill missing historical values from reliable sources where needed
- create clean, stable final tables for Power BI import

**Output: Clean, validated Excel/CSV tables that become the official inputs to Power BI.**
![Cleaned Data Excel](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/cleaned%20data%20excel.png)

### Step 4: Power BI Data Model

A star-style model was implemented to ensure:
- consistent filtering across pages
- stable measures and ranking logic
- clean drillthrough behavior
  
![Measures](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/measures.png)

**Typical model structure:**
- Fact table: Super Bowl Games (one row per Super Bowl)
- Dimension tables:
- Teams (team names/attributes)
- Dates (year/date)
- Halftime Artists (artist, songs, rating)
- Ads/Product Type (category counts/costs)
- Viewership (metrics by year/SB)
  
![Visualization Setup](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/visualization%20setup.png)
 
**Relationship approach (example):**
- Date[Year] → Games[Year] (1:* single direction)
- Teams[Team] → Games[Winner] and Teams[Team] → Games[Loser] (handled via role-based measures or separate mapping tables)
- Halftime[super_bowl] → Games[SB] (1:* or 1:1 depending on structure)
- Viewership[super_bowl] → Games[SB]

### Step 5: Calculated Columns (Examples)
Calculated columns were used to support visuals and sorting.

*Example 1: Points Difference*
```
Points Difference =
'super_bowls_final'[winner_pts] - 'super_bowls_final'[loser_pts]
```
*Example 2: Combined Points*
```
Combined Points =
'super_bowls_final'[winner_pts] + 'super_bowls_final'[loser_pts]
```
### Step 6: DAX Measures (Examples)

Measures were created for KPIs, rankings, and comparisons across dashboards.
*DAX Measure 1: Total Super Bowl Viewership*
```
Total Super Bowl Viewership =
SUM ( 'viewership_final'[viewership] )
```
*DAX Measure 2: Total In-Stadium Attendance*
```
Total In-Stadium Attendance =
SUM ( 'super_bowls_final'[attendance] )
```
*DAX Measure 3: MVP (Selected Year)*
```
MVP (Selected Year) =
SELECTEDVALUE ( 'super_bowls_final'[mvp] )
```
### Step 7: Rankings (Top/Bottom Logic)

To correctly show Top 5 and Bottom 5 lists, ranking measures were used (not alphabetical ranking).
*DAX Measure: Avg Performance Rating (Artist-Year)*

```
Avg Performance Rating (Artist-Year) =
AVERAGE ( 'halftime_musician_final'[performance_rating] )
```
*DAX Measure: Artist Rank (High is Best)*

```
Artist Rank (Best) =
RANKX (
    ALL ( 'halftime_musician_final'[artist] ),
    [Avg Performance Rating (Artist-Year)],
    ,
    DESC,
    Dense
)
```
![Dashboard Setup](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/Dashboard%20setup.png)

*DAX Measure: Artist Rank (Low is Worst)*

```
Artist Rank (Worst) =
RANKX (
    ALL ( 'halftime_musician_final'[artist] ),
    [Avg Performance Rating (Artist-Year)],
    ,
    ASC,
    Dense
)
```
How to use these in visuals:

Top 5 Artists visual:
- Visual filter: Artist Rank (Best) is less than or equal to 5
  - Bottom 5 Artists visual:

- Visual filter: Artist Rank (Worst) is less than or equal to 5
  -This prevents the Top 5 and Bottom 5 from returning the same list.

## Final Visualizations
### Game Overview Page
![Game Overview Drill 2](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/Game%20Overview%20Dashboard%20Drill%202.png)

**Highlights:**

- combined points, stadium attendance, network, MVP, point difference
- top/bottom 5 teams
- score trends and point distribution by state
- drill-through exploration by year
![Game Overview Drill](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/Game%20Overview%20Dashboard%20Drill.png)


### Viewership Comparison Page
**Highlights:**
![Viewership Dashboard Main](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/Viewership%20Dashboard%20Main.png)

- household share, total viewership, ad cost, scatter plot (attendance vs viewership)
- top/bottom year stats for viewership
- Halftime Review Page
  
![Viewership Dashboard Drill](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/Viewership%20Dashboard%20Drill.png)
![Viewership Dashboard Drill 2](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/Viewership%20Dashboard%20Drill%202.png)

**Highlights:**

- best and worst artist cards
- top rating by artist and bottom rating by artist
- top and bottom ads by product type

![Halftime Dashboard Main](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/Halftime%20Dashboard%20Main.png)

**Key Insights (Examples)**

- performance and audience metrics shift across eras, with clear spikes and dips across decades
- some halftime artists consistently rank high by performance rating while others fall to the bottom
- ad product categories show dominance patterns (top categories) and minimal presence areas (bottom categories)

![Halftime Dashboard Drill](https://github.com/iyanusol/Superbowl-50-Year-Game-Analysis/blob/main/images/Halftime%20Dashboard%20Drill.png)

**Tools Used**

- Python (pandas): cleaning, standardization, merge/append workflows
- Excel: consolidation, validation, historical completion, final QA
- Power BI: data modeling, relationships, DAX measures, interactive dashboards


### Author
**Iyanu Adebara**
LinkedIn: ![LinkedIn](https://www.linkedin.com/in/iyanu-adebara-5a62a0103/)
