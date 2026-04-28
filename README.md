# National Parks Visitor Spending Analysis
### MIST 4610 — Group Project 2 

> *How does visitor type shape economic impact across America's national parks — and which parks are leaving value on the table?*

---

## Team Name & Members

**Team Name:** *Group 8*

| Name | GitHub |
|------|--------|
| *Connor Hofmann* | *(link)* |
| *Taylor Keller* | *(link)* |
| *Josie Bowden* | *(link)* |
| *Dean Wadud* | *(link)* |
| *Tyler Pawlowski* | *(link)* |

---

## Project Overview

America's 63 national parks generate **$26.5 billion+ in annual visitor spending** — but raw visitation numbers are a poor proxy for economic output. This project investigates how visitor *type* (not just visitor *count*) drives economic impact, which parks under-leverage high-value visitor segments, and what seasonal patterns reveal about untapped opportunity.

We built an interactive Tableau dashboard using three National Park Service datasets sourced from [data.gov](https://catalog.data.gov/dataset) to answer two interconnected analytical questions.

| Metric | Value |
|--------|-------|
| National Parks analyzed | 63 |
| Protected acres | 85M+ |
| Annual visitors | 325M+ |
| Annual visitor spending | $26.5B+ |

---

## Dataset Description

All source data was obtained from the **National Park Service (NPS)** via [data.gov](https://catalog.data.gov/dataset) and the NPS Social Science program.

### Source Files

| File | Source | Rows (approx.) | Key Columns | Used In |
|------|--------|----------------|-------------|---------|
| `spending_profiles.csv` | NPS Visitor Spending Effects (VSE) | ~500,000+ | `park_code`, `year`, `months`, `profile_name`, `segment`, `spending_category`, `spending_group`, `reporting_group`, `spending_pppdn` | Q1.1 |
| `segment_shares_monthly.csv` | NPS Visitor Use Statistics | ~50,000+ | `park_code`, `year`, `month`, `segment`, `segment_split_2024` | Q1.2, Q2.2 |
| `Park_Rankings_Merged.xlsx` | NPS Annual Visitation Reports | ~63 | `Park Code`, `Park Name`, `Visitor Rank`, `Recreation Visitors`, `State`, `Acres` | Context / joins |
| `nps_analysis_output.csv` | **Derived** (see note below) | ~63 | `park_code`, `park_name`, `visitor_rank`, `hv_share`, `hv_concentration`, `efficiency_rank`, `quadrant` | Q2.1, Q2.2 |

### Column Descriptions

**`spending_profiles.csv`** — Per-person-per-day spending by visitor segment and spending category. The key metric `spending_pppdn` is the dollar amount an average visitor in that segment spends per day in that category. Spending categories include: hotels, restaurants, gas, groceries, camping fees, local transportation, recreation & entertainment, and souvenirs & retail. Segments represent visitor type: `LodgeIN` (in-park lodge guest), `LodgeOut` (outside-park lodge), `CampIN` (in-park camper), `CampOut` (outside-park camper), `NLDay` (non-local day visitor), `LocalDay` (local day visitor), `Other`, and `Backcountry`.

**`segment_shares_monthly.csv`** — The proportion of visitors at each park that fall into each segment, broken out by park, year, and month. `segment_split_2024` is a decimal between 0 and 1 representing that segment's share of total visitors for a given park-month combination. All segments for a given park-month sum to 1.0.

**`Park_Rankings_Merged.xlsx`** — Park-level reference data. Includes official NPS visitor rank (1 = most visited), total recreation visitor count, state, and acreage. Used to provide geographic and scale context for visualizations.

> **⚠️ Note on `nps_analysis_output.csv`:** This file was pre-computed from the three NPS source files above using Python (pandas). It aggregates park-level high-value segment shares (`LodgeIN` + `CampIN`), seasonal concentration indices, economic efficiency scores, and assigns each park a quadrant classification for Q2. **No external data was introduced** — this is a derived summary of the original NPS sources, included to simplify Tableau calculations and avoid limitations with complex multi-table aggregations inside Tableau. All underlying values trace directly back to the three source datasets.

---

## Questions

### Question 1
**"How do visitor spending patterns and seasonal composition vary across parks, and what does this reveal about the economic impact of different visitor types?"**

**Why it matters:** National parks are publicly managed resources, and understanding *which types of visitors* drive the most economic value — not just how many visitors show up — has significant implications for infrastructure investment, concessionaire strategy, and community economic planning. A park serving 10 million local day visitors may generate less regional economic impact than one serving 1 million overnight lodge guests. This question quantifies that gap and maps it seasonally.

**Data connection:** Uses `spending_pppdn` from `spending_profiles.csv` to benchmark segment economics, and `segment_split_2024` from `segment_shares_monthly.csv` to show how park visitor composition shifts month by month.

---

### Question 2
**"Which parks are under-leveraging high-value visitor segments — and is it a seasonality problem or a structural one?"**

**Why it matters:** Identifying *which* parks fail to capture high-value overnight visitors — and whether that failure is a seasonal pattern (fixable via promotions, capacity extension) or a structural constraint (proximity to urban areas, no in-park lodging, geography) — directly informs how the NPS and surrounding communities should prioritize investment. A park that underperforms only in winter has a different problem than one that underperforms year-round.

**Data connection:** Uses the derived `nps_analysis_output.csv` for park-level efficiency classification, cross-referenced against the monthly segment share data from `segment_shares_monthly.csv` and visitor volume from `Park_Rankings_Merged.xlsx`.

---

## Data Manipulations & Calculations

### Pre-Processing (Python — `nps_analysis_output.csv`)
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

---

## Analysis & Results

### Q1.1 — Spending Composition by Visitor Segment

**Visualization:** Dual-axis bar chart — 100% stacked bars showing *where* money goes (by reporting group), with a secondary axis showing absolute $/pppd totals per segment.

*(Insert Tableau screenshot here)*

**Results:**
- **In-Park Lodge Guests (`LodgeIN`)** generate the highest daily spend — lodging alone accounts for 50–60% of their daily expenditure, and their total $/pppd is **5–8× that of a Local Day Visitor**.
- **Non-Local Day Visitors** spend 3× more per day than Local Day Visitors, driven by gas, restaurants, and retail.
- **Local Day Visitors** have the lowest absolute spend but represent the highest share of visitors at many high-traffic, gateway-city-adjacent parks.
- Segment composition — not total visitor count — is the dominant predictor of park economic output.

---

### Q1.2 — Monthly Visitor Segment Mix by Park

**Visualization:** 100% stacked area chart by month, filterable by park. Highlights `LodgeIN` and `CampIN` (high-value) versus `NLDay`, `LocalDay`, and `Other` (lower-value) segments.

*(Insert Tableau screenshot here)*

**Results:**
- **Joshua Tree (JOTR):** Dominated by `NLDay` and `LocalDay` year-round. Near-zero in-park overnight presence in all 12 months — an inherently low-efficiency visitor mix regardless of season.
- **Glacier (GLAC):** Sharp spike in `LodgeIN` + `CampIN` in July–August, falling to near-zero in off-months. High peak-season efficiency, but extreme concentration risk.
- **The contrast operationalizes Q1.1:** Glacier's July visitor is worth 5–8× Joshua Tree's typical visitor in daily economic terms — despite both parks drawing large crowds.

---
