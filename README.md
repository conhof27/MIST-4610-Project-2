# National Parks Visitor Spending Analysis
### MIST 4610 — Group Project 2 

> *How does visitor type shape economic impact across America's national parks — and which parks are leaving value on the table?*

## Team Name & Members

**Team Name:** *Group 8*

| Name | GitHub |
|------|--------|
| *Connor Hofmann* | *[Project 2](https://github.com/conhof27/MIST-4610-Project-2/tree/main)* |
| *Taylor Keller* | *(link)* |
| *Josie Bowden* | *(link)* |
| *Dean Wadud* | *(link)* |
| *Tyler Pawlowski* | *(link)* |

## Project Overview

America's 63 national parks generate **billions** in annual visitor spending, but raw visitation numbers are a poor proxy for economic output and impact.  This project looks into how visitor *type* (not just visitor *count*) drives economic impact, which parks under-leverage high-value visitor segments, and what seasonal patterns reveal about potential untapped opportunities.

We built Tableau visualizations using three National Park Service datasets sourced from [data.gov](https://catalog.data.gov/dataset) to answer two interconnected questions.

## Dataset Description

All source data was obtained from the **National Park Service (NPS)** via [data.gov](https://catalog.data.gov/dataset).

### Source Files

| File | Source | Rows (approx.) | Key Columns | Used In |
|------|--------|----------------|-------------|---------|
| `spending_profiles.csv` | NPS Visitor Spending Effects | ~500,000+ | `park_code`, `year`, `months`, `profile_name`, `segment`, `spending_category`, `spending_group`, `reporting_group`, `spending_pppdn` | Q1.1 |
| `segment_shares_monthly.csv` | NPS Visitor Use Statistics | ~50,000+ | `park_code`, `year`, `month`, `segment`, `segment_split_2024` | Q1.2, Q2.2 |
| `Park_Rankings_Merged.csv` | NPS Annual Visitation Reports | ~63 | `Park Code`, `Park Name`, `Visitor Rank`, `Recreation Visitors`, `State`, `Acres` | Context / joins |
| `nps_analysis_output.csv` | **Derived** (see note below) | ~63 | `park_code`, `park_name`, `visitor_rank`, `hv_share`, `hv_concentration`, `efficiency_rank`, `quadrant` | Q2.1, Q2.2 |

### Column Descriptions

**`spending_profiles.csv`** — Per-person-per-day spending by visitor segment and spending category. The key metric `spending_pppdn` is the dollar amount an average visitor in that segment spends per day in that category. Spending categories include: hotels, restaurants, gas, groceries, camping fees, local transportation, recreation & entertainment, and souvenirs & retail. Segments represent visitor type: `LodgeIN` (in-park lodge guest), `LodgeOut` (outside-park lodge), `CampIN` (in-park camper), `CampOut` (outside-park camper), `NLDay` (non-local day visitor), `LocalDay` (local day visitor), and `Other`.

**`segment_shares_monthly.csv`** — The proportion of visitors at each park that fall into each segment, broken out by park, year, and month. `segment_split_2024` is a decimal between 0 and 1 representing that segment's share of total visitors for a given park-month combination. All segments for a given park-month sum to 1.

**`Park_Rankings_Merged.csv`** — Park-level reference data used to provide context for visualizations.  This dataset was cleaned to provide better details into basic park information, such as: coordinates, # of visitors, state, and acreage.

**`nps_analysis_output.csv`:** This file was computed and cleaned from the three NPS source files above using an LLM. It aggregates park-level high-value segment shares (`LodgeIN` + `CampIN`), seasonal concentration indices, economic efficiency scores, and assigns each park a quadrant classification for Q2. **No external data was introduced** — this is a derived summary of the original NPS sources, included to simplify Tableau calculations and avoid previous errors when trying to work with complex multi-table aggregations inside Tableau.


## Questions

### Question 1
**"How do visitor spending patterns and seasonal composition vary across parks, and what does this reveal about the economic impact of different visitor types?"**

**Why it matters:** National parks are managed by the government, and understanding which types of visitors drive the most economic value has significant implications for investment (gov. spending), concession strategy, and community economic planning. A park serving 10 million local day visitors may generate less regional economic impact than one serving 1 million overnight lodge guests. This question looks into that gap and maps it seasonally.

**Data connection:** Uses `spending_pppdn` from `spending_profiles.csv` to benchmark segment economics, and `segment_split_2024` from `segment_shares_monthly.csv` to show how park visitor composition shifts month by month.

### Question 2
**"Which parks are under-leveraging high-value visitor segments and is it a seasonality problem or a structural one?"**

**Why it matters:** Identifying which parks fail to capture high-value overnight visitors and whether that failure is a seasonal pattern or a structural problem (proximity to urban areas, no in-park lodging, geography) directly informs how the NPS and surrounding communities should prioritize investment. A park that underperforms only in winter has a different problem than one that underperforms year-round.

**Data connection:** Uses the derived `nps_analysis_output.csv` for park-level efficiency classification, cross-referenced against the monthly segment share data from `segment_shares_monthly.csv` and visitor volume from `Park_Rankings_Merged.xlsx`.

## Data Manipulations & Calculations

### Pre-Processing (LLM — `nps_analysis_output.csv`)
- **High-Value Share (`hv_share`):** For each park, averaged the monthly `segment_split_2024` values for `LodgeIN` and `CampIN` across all 12 months of 2024. This produces a single number representing what proportion of that park's typical visitor base are high-value overnight guests.
- **Seasonal Concentration Index (`hv_concentration`):** Calculated as the standard deviation of monthly high-value share across months 1–12. High values indicate boom-bust seasonal patterns; low values indicate year-round consistency.
- **Economic Efficiency Rank (`efficiency_rank`):** Parks were ranked by `hv_share` descending (1 = highest high-value share). Combined with `visitor_rank` from the NPS visitation data to enable scatter plot positioning in Q2.1.
- **Quadrant Classification (`quadrant`):** Each park was assigned one of four labels based on `hv_share` (above/below median) and `hv_concentration` (above/below median):
  - `Year-Round Premium` — high hv_share, low concentration (consistent overnight visitors)
  - `Destination Boom-Bust` — high hv_share, high concentration (compressed season)
  - `Seasonal Day-Trip` — moderate hv_share only in peak months
  - `Structural Underperformer` — low hv_share across all months

### Inside Tableau
- **Q1.1:** Segment totals computed as SUM of `spending_pppdn` per `reporting_group`, then averaged across parks using Tableau's default aggregation. A calculated field `Total $/pppd` sums all category values per segment for the bar height on the secondary axis.
- **Q1.2:** `segment_split_2024` filtered to `LodgeIN` and `CampIN` segments only. Month converted to discrete dimension (1–12). Park filter applied via parameter to enable switching between parks.
- **Q2.1:** `visitor_rank` and `efficiency_rank` placed on X/Y axes. `quadrant` used as the Color mark. Park Name on Detail for hover tooltip.
- **Q2.2:** Park rows × Month columns heatmap. Color = AVG(`segment_split_2024`) filtered to LodgeIN + CampIN. Rows sorted by `hv_share` descending and grouped by `quadrant`.


## Analysis & Results

### Q1.1 — Spending Composition by Visitor Segment

**Visualization:** Dual-axis bar chart — stacked bars showing *where* money goes (by reporting group), with a secondary axis showing absolute $/pppd totals per segment.

<img width="1506" height="245" alt="Screenshot 2026-04-28 at 3 26 58 PM" src="https://github.com/user-attachments/assets/ffe1f858-df6a-4a12-918d-bd374b161e66" />


**Results:**
- **In-Park Lodge Guests (`LodgeIN`)** generate the highest daily spend — lodging alone accounts for 50–60% of their daily expenditure, and their total $/pppd is **5–8× that of a Local Day Visitor**.
- **Non-Local Day Visitors** spend 3× more per day than Local Day Visitors, driven by gas, restaurants, and retail.
- **Local Day Visitors** have the lowest absolute spend but represent the highest share of visitors at many high-traffic, gateway-city-adjacent parks.
- Segment composition — not total visitor count — is the dominant predictor of park economic output.


### Q1.2 — Monthly Visitor Segment Mix by Park

**Visualization:** Stacked area chart by month, filterable by park. Highlights `LodgeIN` and `CampIN` (high-value) versus `NLDay`, `LocalDay`, and `Other` (lower-value) segments.

<img width="1508" height="909" alt="Screenshot 2026-04-28 at 3 28 37 PM" src="https://github.com/user-attachments/assets/4fe1d92c-04b9-4fb7-9b82-ac020327bce8" />


**Results:**
- **Great Smoky Mountains NP** — the most visited park in America — shows near-zero In-Park Lodge Guest and In-Park Camper presence across all 12 months. The stack is almost entirely gray year-round, confirming this is a structural deficit, not a seasonal one. Despite 11M+ annual visitors, virtually none are generating high-value overnight economic activity inside the park.
- **Glacier NP** shows a dramatic and narrow spike in both In-Park Lodge Guests and In-Park Campers concentrated in months 6–9 (June–September), with essentially no high-value visitor presence outside that window. Its economic output is powerful but compressed into roughly 8 weeks — making it highly vulnerable to any disruption of peak season.
- **Yosemite NP** is the strongest performer of the three — maintaining meaningful In-Park Lodge Guest share (dark teal) across nearly all 12 months, with In-Park Camper share (dark green) rising through the spring and summer. Its high-value visitor presence is both substantial and more evenly distributed than Glacier's.
- **Summary** a structural underperformer with no overnight base (Smoky Mountains), a boom-bust destination dependent on a single summer window (Glacier), and a year-round premium park that consistently captures high-value overnight visitors (Yosemite). These three parks directly set up the classification framework in Q2.
  

### Q2.1 — Visitor Rank vs. Economic Efficiency 

**Visualization:** Scatter plot showing each park's visitor rank (x-axis) against its economic efficiency rank (y-axis), with parks color-coded by quadrant classification. A linera trendline shows where visitor rank and efficiency rank are equal — parks above the line over-deliver economically relative to their traffic; parks below under-deliver.

<img width="1132" height="780" alt="Screenshot 2026-04-28 at 6 38 27 PM" src="https://github.com/user-attachments/assets/b674987e-6c43-4a9d-9964-9bbc41f7918c" />

**Results:**
- **High traffic does not equal high economic value.** Great Smoky Mountains NP (Visitor Rank #1) falls well below the trendline line despite being the most visited park in America.  It ranks poorly on economic efficiency due to near-zero in-park overnight visitor share. Cuyahoga Valley NP shows the same pattern.
- **Channel Islands NP and several remote western parks punch far above their weight.** With low visitor volumes but high proportions of overnight in-park guests, these parks generate disproportionate economic output relative to their traffic rank, placing them above the trendline line in the upper-left region.
- **Yosemite and Grand Canyon** sit near or above the parity line despite being among the top 5 most visited parks, confirming their status as `Year-Round Premium` parks that successfully convert high traffic into high-value overnight stays.
- **Quadrant color-coding reveals structural vs. seasonal patterns at a glance.** `Structural Underperformer` parks (red) cluster below the line regardless of traffic volume, while `Destination Boom-Bust` parks (blue) show wide scatter — indicating inconsistent efficiency tied to compressed seasons.


### Q2.2 — Park × Month Heatmap 

**Visualization:** Heatmap with parks as rows (grouped and sorted by quadrant), months as columns, and cell color representing the average high-value visitor share (LodgeIN + CampIN) for that park-month combination. Color scale runs from gold (low) to deep green (high). White cells indicate months with no recorded overnight lodge or camp visitor data.

<img width="1501" height="904" alt="Screenshot 2026-04-28 at 6 39 32 PM" src="https://github.com/user-attachments/assets/1021c97e-45f9-4022-abd9-21a7fde4a869" />

**Results:**
- **Year-Round Premium parks** (top group) show consistent light-to-medium green shading across all 12 months, confirming stable overnight visitor presence regardless of season. Channel Islands NP and Great Basin NP maintain meaningful high-value share even in January and February.
- **Destination Boom-Bust parks** reveal the most striking patterns — North Cascades, Voyageurs, Isle Royale, and the Alaska wilderness parks (Gates of the Arctic, Lake Clark, Katmai, Wrangell-St. Elias) show deep green spikes in 1–3 summer months surrounded by white or near-zero cells. Their entire economic output is compressed into a narrow window, making them highly vulnerable to weather disruptions, budget cuts, or access changes during peak season.
- **Seasonal Day-Trip parks** (bottom group) show mild, diffuse summer bumps but never reach the deep greens seen in the Boom-Bust category — confirming that their visitor base skews toward lower-value day visitors even during peak months.
- **Together, 2.1 and 2.2 answer the question directly:** parks that underperform economically (2.1) either lack overnight infrastructure year-round (structural) or compress all high-value activity into a few summer weeks (seasonal). The fix for each type is fundamentally different — one requires infrastructure investment, the other requires capacity extension and off-season programming.

## Tableau Package & Workbook

**LLM Usage Statement:** Generative AI was used in collaboration with our teams findings to not only help clean and decipher thousands of rows of data in our sources but to also refine and strengthen the explaination of our initial analysis.  
