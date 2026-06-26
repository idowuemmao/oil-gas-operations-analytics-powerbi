# ⚡ Oil, Gas & Energy Operations Analytics — Power BI Report

> A 3-page executive-grade Power BI report covering production performance, operational risk, maintenance efficiency, safety compliance, sustainability targets, and financial impact across a multi-site Oil, Gas & Energy portfolio.

---

## 📌 Repository Description

**oil-gas-operations-analytics-powerbi** — Executive Power BI report for upstream oil & gas operations. Tracks production vs. target, capacity utilization, downtime-driven revenue loss, safety incidents, maintenance efficiency, CO₂ emissions intensity, and energy performance across offshore platforms, refineries, onshore sites, and LNG terminals. Built for operations managers, HSE leads, sustainability officers, and C-suite decision-makers.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Business Objectives](#business-objectives)
- [Dataset Overview](#dataset-overview)
- [Data Model](#data-model)
- [Report Architecture](#report-architecture)
  - [Page 1 — Executive Summary (Production / Revenue Toggle)](#page-1--executive-summary)
  - [Page 2 — Operational Risk & Maintenance](#page-2--operational-risk--maintenance)
  - [Page 3 — Efficiency & Environmental Cost](#page-3--efficiency--environmental-cost)
  - [Tooltip Pages](#tooltip-pages)
- [DAX Measures](#dax-measures)
- [KPI Reference](#kpi-reference)
- [Design Decisions](#design-decisions)
- [How to Use This Report](#how-to-use-this-report)
- [Stakeholder Guide](#stakeholder-guide)
- [Tools & Technologies](#tools--technologies)
- [Author](#author)

---

## Project Overview

This Power BI report was built as a structured analytics challenge project simulating real-world energy operations intelligence. It models the kind of operational analytics solution a senior BI consultant would deliver to an oil & gas operator managing multiple sites across regions and countries.

The report answers three escalating business questions across its three pages:

1. **Are we hitting our production and revenue targets — and where are we falling short?**
2. **Where are the operational risks, and what is their safety and maintenance footprint?**
3. **What is the efficiency and environmental cost of how we operate — and is it financially sustainable?**

Each page is designed to stand alone for its target audience while linking to the others through consistent slicers, shared KPI logic, and drill-through navigation.

---

## Business Objectives

| Objective | Page |
|---|---|
| Monitor production and revenue performance vs. targets across all sites | Page 1 |
| Identify underperforming sites, regions, and site types | Page 1 |
| Quantify the financial cost of downtime and production gaps | Page 1 (Revenue toggle) |
| Track safety incidents against tolerance thresholds by site and month | Page 2 |
| Evaluate whether maintenance spend is translating to operational reliability | Page 2 |
| Assess composite operational risk across downtime, safety, and cost dimensions | Page 2 |
| Monitor CO₂ emissions against annual targets and identify intensity outliers | Page 3 |
| Analyse energy efficiency trends across site types | Page 3 |
| Connect oil price volatility to maintenance cost exposure | Page 3 |
| Enable executive-level fleet-wide target compliance at a glance | All pages |

---

## Dataset Overview

The dataset covers a multi-site Oil, Gas & Energy operation across five tables:

### DimSites
| Column | Description |
|---|---|
| SiteID | Unique site identifier |
| SiteName | Operational name of the site |
| Region | Geographic region |
| Country | Country of operation |
| SiteType | Offshore Platform, Refinery, Onshore, LNG Terminal |
| Capacity_BPD | Maximum production capacity in barrels per day |
| Latitude | Geographic coordinate |
| Longitude | Geographic coordinate |

### DimDates
| Column | Description |
|---|---|
| Date | Full date |
| Year | Calendar year |
| Month | Month number |
| MonthName | Month name |
| Quarter | Q1–Q4 |
| Weekday | Day of week |

### FactOperation
| Column | Description |
|---|---|
| Date | Daily grain |
| SiteID | FK to DimSites |
| Production_Barrels | Daily barrels produced |
| OilPrice_USD | Daily oil price per barrel |
| Revenue_USD | Daily revenue |
| Downtime_Hours | Downtime hours recorded |
| Safety_Incidents | Incident count |
| Energy_Consumption_MWh | Energy consumed |
| CO2_Emissions_Tons | CO₂ emitted |

### FactMaintenance
| Column | Description |
|---|---|
| MaintenanceID | Unique maintenance record ID |
| SiteID | FK to DimSites |
| MaintenanceMonth | Month of maintenance activity |
| PlannedMaintenanceHours | Scheduled hours |
| MaintenanceCost_USD | Total maintenance cost |
| TechniciansAssigned | Headcount assigned |

### KPIMonthlySummary
Monthly grain target vs. actual table used for all target tracking and variance calculations.

| Column | Description |
|---|---|
| Date | Month-end date |
| SiteID | FK to DimSites |
| Production_Barrels | Monthly actual production |
| Revenue_USD | Monthly actual revenue |
| Downtime_Hours | Monthly actual downtime |
| Safety_Incidents | Monthly actual incidents |
| CO2_Emissions_Tons | Monthly actual CO₂ |
| Production_Target_Barrels | Monthly production target |
| Revenue_Target_USD | Monthly revenue target |
| Downtime_Target_Hours | Monthly downtime tolerance |
| Safety_Incident_Target | Monthly safety tolerance |
| CO2_Emissions_Target_Tons | Monthly CO₂ budget |
| Production_vs_Target_Pct | Variance % |
| Revenue_vs_Target_Pct | Variance % |
| Downtime_vs_Target_Pct | Variance % |
| CO2_vs_Target_Pct | Variance % |
| Safety_vs_Target_Pct | Variance % |
| Safety_Target_Status | Pass / Fail label |

> **Important:** Safety_Incident_Target represents a maximum tolerance threshold. Lower actual incidents are always better. Safety_vs_Target_Pct = (Target − Actual) ÷ Target. A positive value means the site is within tolerance.

---

## Data Model

The report uses a **star schema** with the KPIMonthlySummary table as a second fact table bridged through DimSites and DimDates.

```
DimDates  ──────────────┬──── FactOperation
                        │           │
                        │      (SiteID, Date)
                        │           │
DimSites  ──────────────┼──── FactMaintenance
                        │           │
                        │      (SiteID, MaintenanceMonth)
                        │           │
                        └──── KPIMonthlySummary
                                   │
                              (SiteID, Date)
```

**Relationships:**
- `DimDates[Date]` → `FactOperation[Date]` (many-to-one)
- `DimDates[Date]` → `KPIMonthlySummary[Date]` (many-to-one, month-end grain)
- `DimSites[SiteID]` → `FactOperation[SiteID]` (many-to-one)
- `DimSites[SiteID]` → `FactMaintenance[SiteID]` (many-to-one)
- `DimSites[SiteID]` → `KPIMonthlySummary[SiteID]` (many-to-one)

---

## Report Architecture

### Page 1 — Executive Summary

**Toggle:** Production view | Revenue view (controlled by a button-based parameter)

The page is a toggle that replaces all production-specific metrics and visuals with their revenue equivalents at the click of a button. Both views share the same layout, slicers, and navigation.

#### Slicers
· Site Type · Capacity Tier

---

#### Production View

**KPI Cards**

| KPI | Reference Insight |
|---|---|
| Total Production Barrels | Volume delivered this period vs. prior period |
| Capacity Utilization % | Fleet-wide utilisation against installed capacity |
| Total Downtime Hours | Cumulative hours lost across all sites |
| Production Gap Value (USD) | Financial cost of the production shortfall at current oil price |
| Sites Meeting All Targets | Count of sites compliant across all 5 target dimensions |

**Visuals**

| # | Visual | Business Question Answered |
|---|---|---|
| 1 | Column Chart  with Ref Line + line combo — Production Target (Ref Line), Total Production (Columnn Chart), Prod vs. Target % (Secondary line) by Month-Year | Is production improving or deteriorating month-on-month? Are we above or below target consistently? |
| 2 | Scatter plot — Utilization % vs. Production Gap by Site Name, legend: Site Type | Which specific sites are underutilising capacity AND generating the largest production gaps? |
| 3 | Bar chart — Capacity Utilization % by Region (drill down to Country) with 80% reference line | Which regions are operating efficiently? Which are structurally below the utilisation floor? |
| 4 | Heatmap — Site Name × Target Dimension (Production, Revenue, Downtime, CO₂, Safety, Utilization). Dark green = above target, light green = within 5%, pink = below target | At a glance: which sites are multi-dimensional underperformers requiring priority intervention? |

<img width="1019" height="571" alt="P1-1" src="https://github.com/user-attachments/assets/7150046a-9047-4e49-bf27-f95955058f33" />

---

#### Revenue View

**KPI Cards**

| KPI | Reference Insight |
|---|---|
| Total Revenue | Revenue generated this period vs. prior period |
| Capacity Utilization % | Unchanged from production view |
| Total Downtime Hours | Unchanged from production view |
| Production Gap Value (USD) | Unchanged from production view |
| Sites Meeting All Targets | Unchanged from production view |

**Visuals**

| # | Visual | Business Question Answered |
|---|---|---|
| 1 |  Column Chart  with Ref Line + line combo — Revenue Target (Ref Line), Total Revenue (Columnn Chart), Rev vs. Target % (Secondary line) by Month-Year | Is revenue tracking ahead or behind target? Where are the most significant monthly misses? |
| 2 | Scatter plot — Utilization % vs. Revenue Gap by Site Name, legend: Site Type | Which sites are leaving the most revenue on the table relative to their capacity? |
| 3 | Bar chart — Capacity Utilization % by Region (drill down to Country) with 80% reference line | Same as production view — maintains context continuity across toggle |
| 4 | Waterfall chart — Revenue gap breakdown by root cause driver | What specifically is causing the revenue miss: production volume, downtime hours, oil price variance, or maintenance overspend? |

<img width="1019" height="573" alt="P1-2" src="https://github.com/user-attachments/assets/cfd7f38b-ded5-4cb8-96c4-ac8368f1aaec" />

---

### Page 2 — Operational Risk & Maintenance

**Page objective:** Identify which sites carry the highest operational risk, quantify the financial and safety exposure, and evaluate whether maintenance spend is converting to operational reliability.

#### Slicers
· Site Type · Capacity Tier · Year · Region

#### KPI Cards

| KPI | Reference Insight |
|---|---|
| Revenue at Risk (USD) | Downtime hours × average daily production rate × oil price — the financial cost of lost availability |
| Total Maintenance Cost | Cumulative maintenance spend this period |
| Unplanned Maintenance Ratio | Unplanned ÷ total maintenance hours — high ratio signals reactive rather than preventive culture |
| Incident Free Sites | Count of sites with zero safety incidents in the selected period |
| Total Safety Incidents | Fleet-wide incident count vs. prior period (shown as absolute integer delta, not %) |

**Visuals**

| # | Visual | Business Question Answered |
|---|---|---|
| 1 | Column + line combo — Revenue at Risk (column), Composite Risk Score (line) by Site Name | Which sites combine the highest financial exposure with the highest multi-dimensional risk score? |
| 2 | Heatmap matrix — Row: Site Name · Column: Month Name · Value: Total Safety Incidents. Conditional format: green ≤ target, red > target | In which months and at which sites did safety incidents exceed tolerance? Is there a seasonal or structural pattern? |
| 3 | Scatter plot — Total Downtime Hours vs. Total Maintenance Cost by Site Name, legend: Site Type | Is maintenance spend suppressing downtime? Sites with high cost and high downtime are the efficiency failures. |
| 4 | Dynamic text card — Conclusion | Auto-written insight summarising the page's key risk findings based on current filter context |
| 5 | Dynamic text card — Recommended Action | Auto-written prioritised action derived from the data in context |

<img width="1027" height="576" alt="P2" src="https://github.com/user-attachments/assets/eff3d876-418a-413d-94bb-6931054860cf" />

---

### Page 3 — Efficiency & Environmental Cost

**Page objective:** Quantify the operational efficiency and environmental cost of the fleet. Connect oil price conditions to maintenance cost exposure. Identify which site types and sites are the most carbon and energy intensive. Answer the question: are we operating sustainably and efficiently enough at current oil prices?

#### Slicers
· Country · Capacity Tier · Region

#### KPI Cards

| KPI | Reference Insight |
|---|---|
| CO₂ Emission Gap to Annual Target | Cumulative tons above or below the annual CO₂ budget — widening gap signals regulatory risk |
| Total Energy Consumption (ref: Energy per Barrel) | Fleet energy volume with intensity per barrel in reference label for efficiency context |
| Maintenance Cost per Downtime Hour | High spend not paired with low downtime reveals maintenance ineffectiveness |
| Total CO₂ Emissions (ref: CO₂ per Barrel) | Absolute volume with per-barrel intensity — connects emissions to production behaviour |
| Total Oil Price (ref: Avg Oil Price) | Planning assumption context — below-target price narrows the tolerance for operational inefficiency |

**Visuals**

| # | Visual | Business Question Answered |
|---|---|---|
| 1 | Column + dual-line — CO₂ Emissions (column), CO₂ Target (reference line), CO₂ vs. Target % (secondary line) by Month-Year | Is the emissions gap seasonal or structural? Which months pushed the fleet furthest above target? |
| 2 | Column + line combo — Avg Oil Price (column), Maintenance Cost per Downtime Hour (secondary line) by Quarter (drill down to Month) | When oil price falls, does maintenance cost per downtime hour rise — and by how much? At what price does inefficiency become financially critical? |
| 3 | Scatter plot — Avg Production vs. CO₂ per Barrel by Site Name, legend: Site Type | Are high-volume producers also the most carbon-intensive? Which sites combine high output with high emissions intensity? |
| 4 | Bar chart — Maintenance Cost per Downtime Hour by Site Type (drill down to Site Name) | Which site types and specific sites spend the most per downtime hour yet still accumulate the most downtime? |
| 5 | Dynamic text card — Conclusion + Recommended Action | Fully dynamic DAX-driven insight responding to all active slicers |

<img width="1023" height="575" alt="P3" src="https://github.com/user-attachments/assets/216f9270-85f3-4e91-a1bf-2c80ae852c6c" />

---

### Tooltip Pages

#### Tooltip 1 — Site Risk Profile
*Triggered by: hovering over a site name on the Page 2 column/risk chart*

| Element | Detail |
|---|---|
| Header | Selected site name, site type, country, region |
| KPI Card | Composite Risk Score |
| KPI Card | Risk Tier (Low / Medium / High / Critical) |
| Multi-row Card | Target compliance status across all 6 dimensions: Production, Revenue, Downtime, CO₂, Safety, Utilization |

**Purpose:** Gives operations managers instant site context without leaving the page — who is this site, how risky is it overall, and which specific targets is it failing?

<img width="940" height="381" alt="Tooltip1" src="https://github.com/user-attachments/assets/8c84af2d-2c42-4d0d-924c-1d71549fda6b" />

---

#### Tooltip 2 — Safety Breakdown
*Triggered by: hovering over a cell in the Page 2 safety heatmap*

| Element | Detail |
|---|---|
| KPI Card | Actual incidents for selected site-month |
| KPI Card | Safety target (tolerance threshold) |
| KPI Card | Vs. Target % |
| KPI Card | Status (Within Tolerance / Exceeding Target) |
| Line chart | Safety vs. Target % trend by Weekday for the selected month |

**Purpose:** Allows HSE managers to see not just that a month exceeded target, but which days of the week the incidents concentrated on — enabling shift pattern and scheduling investigation.

<img width="645" height="353" alt="Tooltip2" src="https://github.com/user-attachments/assets/90b19d6b-bae8-4fb6-b9a3-b2fefe6edb5b" />

---

## DAX Measures

### Core Operational Measures

```dax
Total Production Barrels =
SUM(FactOperation[Production_Barrels])

Total Revenue USD =
SUM(FactOperation[Revenue_USD])

Total Downtime Hours =
SUM(FactOperation[Downtime_Hours])

Total Safety Incidents =
SUM(FactOperation[Safety_Incidents])

Total CO2 Emissions =
SUM(FactOperation[CO2_Emissions_Tons])

Total Energy Consumption MWh =
SUM(FactOperation[Energy_Consumption_MWh])

Avg Oil Price =
AVERAGE(FactOperation[OilPrice_USD])
```

### Capacity & Utilization

```dax
Total Capacity BPD =
SUMX(DimSites, DimSites[Capacity_BPD])

Capacity Utilization Pct =
DIVIDE(
    [Total Production Barrels],
    SUMX(
        VALUES(DimSites[SiteID]),
        RELATED(DimSites[Capacity_BPD])
            * COUNTROWS(RELATEDTABLE(DimDates))
    )
)

Production Gap Value USD =
( [Total Capacity Barrels] - [Total Production Barrels] )
    * [Avg Oil Price]
```

### Target Tracking

```dax
Production Target =
SUM(KPIMonthlySummary[Production_Target_Barrels])

Revenue Target USD =
SUM(KPIMonthlySummary[Revenue_Target_USD])

Downtime Target Hours =
SUM(KPIMonthlySummary[Downtime_Target_Hours])

CO2 Target Tons =
SUM(KPIMonthlySummary[CO2_Emissions_Target_Tons])

Sites Meeting All Targets =
COUNTROWS(
    FILTER(
        VALUES(DimSites[SiteID]),
        CALCULATE(MIN(KPIMonthlySummary[Production_vs_Target_Pct])) >= 1
            && CALCULATE(MIN(KPIMonthlySummary[Revenue_vs_Target_Pct])) >= 1
            && CALCULATE(MIN(KPIMonthlySummary[Downtime_vs_Target_Pct])) >= 0
            && CALCULATE(MIN(KPIMonthlySummary[Safety_vs_Target_Pct])) >= 0
            && CALCULATE(MIN(KPIMonthlySummary[CO2_vs_Target_Pct])) >= 0
    )
)
```

### Financial Risk

```dax
Revenue at Risk USD =
DIVIDE([Total Downtime Hours], 24)
    * AVERAGEX(ALL(DimSites), DimSites[Capacity_BPD])
    * [Avg Oil Price]

Maintenance Cost Per Downtime Hour =
DIVIDE(
    SUM(FactMaintenance[MaintenanceCost_USD]),
    [Total Downtime Hours]
)

Unplanned Maintenance Ratio =
DIVIDE(
    [Total Downtime Hours],
    SUM(FactMaintenance[PlannedMaintenanceHours])
        + [Total Downtime Hours]
)
```

### Efficiency & Sustainability

```dax
CO2 Per Barrel =
DIVIDE([Total CO2 Emissions], [Total Production Barrels])

Energy Intensity MWh Per Barrel =
DIVIDE([Total Energy Consumption MWh], [Total Production Barrels])

CO2 Gap to Annual Target =
SUMX(
    VALUES(DimDates[Date]),
    CALCULATE(SUM(KPIMonthlySummary[CO2_Emissions_Tons]))
        - CALCULATE(SUM(KPIMonthlySummary[CO2_Emissions_Target_Tons]))
)
```

### Risk & Composite Scoring

```dax
Composite Risk Score =
VAR DowntimeScore =
    DIVIDE([Total Downtime Hours], CALCULATE([Total Downtime Hours], ALL(DimSites)))
VAR SafetyScore =
    DIVIDE([Total Safety Incidents], CALCULATE([Total Safety Incidents], ALL(DimSites)))
VAR CostScore =
    DIVIDE(
        SUM(FactMaintenance[MaintenanceCost_USD]),
        CALCULATE(SUM(FactMaintenance[MaintenanceCost_USD]), ALL(DimSites))
    )
RETURN
    ( DowntimeScore * 0.35 ) + ( SafetyScore * 0.35 ) + ( CostScore * 0.30 )

Risk Tier =
VAR Score = [Composite Risk Score]
RETURN
    SWITCH(
        TRUE(),
        Score >= 0.75, "Critical",
        Score >= 0.50, "High",
        Score >= 0.25, "Medium",
        "Low"
    )

Incident Free Sites =
COUNTROWS(
    FILTER(
        VALUES(DimSites[SiteID]),
        CALCULATE(SUM(FactOperation[Safety_Incidents])) = 0
    )
)
```

### Dynamic Insight Measures

```dax
Insight_P3_Conclusion =
VAR AvgOilPrice       = AVERAGE(FactOperation[OilPrice_USD])
VAR CO2Gap            =
    SUMX(
        VALUES(DimDates[Date]),
        CALCULATE(SUM(KPIMonthlySummary[CO2_Emissions_Tons]))
            - CALCULATE(SUM(KPIMonthlySummary[CO2_Emissions_Target_Tons]))
    )
VAR MaintPerDowntime  = [Maintenance Cost Per Downtime Hour]
VAR FleetMaintPerHr   =
    CALCULATE([Maintenance Cost Per Downtime Hour], ALL(DimSites))
VAR CO2PerBbl         = [CO2 Per Barrel]
VAR EnergyPerBbl      = [Energy Intensity MWh Per Barrel]
VAR CO2Dir            = IF(CO2Gap > 0, "above", "below")
VAR MaintSignal       =
    IF(MaintPerDowntime > FleetMaintPerHr,
        "above fleet average — spend is not suppressing downtime.",
        "below fleet average — maintenance is delivering value."
    )
RETURN
"Oil at " & FORMAT(AvgOilPrice, "$#,0.00") & "/bbl narrows the margin for inefficiency. "
    & "CO₂ emissions are " & FORMAT(ABS(CO2Gap), "#,0") & " tons " & CO2Dir & " target. "
    & "At " & FORMAT(CO2PerBbl, "0.000") & " T/bbl intensity and "
    & FORMAT(EnergyPerBbl, "0.000") & " MWh/bbl, "
    & "maintenance cost per downtime hour is " & FORMAT(MaintPerDowntime, "$#,0")
    & " — " & MaintSignal

Insight_P3_Action =
VAR CO2Gap =
    SUMX(
        VALUES(DimDates[Date]),
        CALCULATE(SUM(KPIMonthlySummary[CO2_Emissions_Tons]))
            - CALCULATE(SUM(KPIMonthlySummary[CO2_Emissions_Target_Tons]))
    )
VAR MaintCostPerHr    = [Maintenance Cost Per Downtime Hour]
VAR FleetMaintPerHr   =
    CALCULATE([Maintenance Cost Per Downtime Hour], ALL(DimSites))
VAR EnergyPerBbl      = [Energy Intensity MWh Per Barrel]
VAR FleetEnergyPerBbl =
    CALCULATE([Energy Intensity MWh Per Barrel], ALL(DimSites))
VAR CO2Action =
    IF(CO2Gap > 0,
        "Prioritise emission reduction at sites exceeding CO₂/barrel target before the annual gap compounds further. ",
        "CO₂ is within target — sustain current operating discipline. "
    )
VAR MaintAction =
    IF(MaintCostPerHr > FleetMaintPerHr,
        "Review maintenance scheduling at high cost-per-downtime-hour sites — spend is not converting to reliability. ",
        "Maintenance efficiency is tracking positively — protect current planned maintenance cadence. "
    )
VAR EnergyAction =
    IF(EnergyPerBbl > FleetEnergyPerBbl,
        "Target energy intensity reduction at site type consuming above fleet average MWh/barrel.",
        "Energy intensity is at or below fleet average — monitor for regression."
    )
RETURN CO2Action & MaintAction & EnergyAction
```

---

## KPI Reference

| KPI | Source Table | Type | Direction |
|---|---|---|---|
| Total Production Barrels | FactOperation | Volume | Higher = better |
| Total Revenue USD | FactOperation | Financial | Higher = better |
| Capacity Utilization % | FactOperation + DimSites | Efficiency | Higher = better |
| Total Downtime Hours | FactOperation | Operational | Lower = better |
| Production Gap Value USD | Calculated | Financial risk | Lower = better |
| Sites Meeting All Targets | KPIMonthlySummary | Compliance | Higher = better |
| Revenue at Risk USD | Calculated | Financial risk | Lower = better |
| Total Maintenance Cost | FactMaintenance | Cost | Lower = better (in context) |
| Unplanned Maintenance Ratio | Calculated | Reliability | Lower = better |
| Incident Free Sites | FactOperation | Safety | Higher = better |
| Total Safety Incidents | FactOperation | Safety | Lower = better |
| Composite Risk Score | Calculated | Risk | Lower = better |
| CO₂ Gap to Annual Target | KPIMonthlySummary | Sustainability | Negative = better |
| CO₂ per Barrel | Calculated | Intensity | Lower = better |
| Energy Intensity MWh/bbl | Calculated | Efficiency | Lower = better |
| Maintenance Cost per Downtime Hour | Calculated | Efficiency | Lower = better |
| Avg Oil Price | FactOperation | Market | Context only |

---

## Design Decisions

### Toggle page (Page 1)
A single page uses a field parameter or bookmark-driven button pair to switch all visuals between production and revenue contexts. This avoids page duplication, maintains slicer state, and reduces report navigation friction for executives who move between production and financial views repeatedly.

### Safety delta — absolute not percentage
Safety incident deltas are displayed as integer changes (e.g. "+8 incidents") rather than percentages. A move from 1 to 2 incidents is 100% — a percentage framing trivialises what may be a material safety event. The absolute delta communicates severity more honestly.

### Heatmap for target compliance
The 6-dimension target compliance heatmap (dark green / light green / pink) was chosen over a table with numbers because executives scanning the fleet need pattern recognition, not arithmetic. The colour grammar communicates compliance state faster than any numeric format.

### Conditional formatting on safety matrix
The safety heatmap on Page 2 uses the Safety_Incident_Target from KPIMonthlySummary as its threshold — not a hardcoded number. This means target logic varies correctly by site type and operating risk profile as the data intends.

### Composite Risk Score weighting
Downtime 35% · Safety 35% · Cost 30%. Safety and downtime are weighted equally as the primary operational risk indicators. Cost is a consequence and an amplifier, not a cause — hence the lower weight. All inputs are normalised to fleet totals using ALL(DimSites) to produce comparable relative scores.

### Tooltip pages
Both tooltip pages are designed to extend, not repeat, the information visible on the parent chart. Tooltip 1 adds site context and multi-KPI compliance that would require a full drill-through to access otherwise. Tooltip 2 adds weekday granularity to the monthly safety heatmap — enabling a level of root-cause investigation not possible from the heatmap cell alone.

---

## How to Use This Report

### Navigation
Three main pages are accessible via the navigation bar at the top of every page. Page 1 has an additional internal toggle (Production | Revenue) controlled by buttons on the canvas.

### Filtering
All slicers are persistent within a page. Cross-page filtering is not active by default — each page maintains its own filter context. Use the Reset Page button on Page 1 to clear all active filters.

### Drill-downs
- **Page 1 bar chart:** Click the drill-down arrow to move from Region to Country level
- **Page 3 bar chart:** Click the drill-down arrow to move from Site Type to individual Site Name
- **Page 3 line chart:** Click to drill from Quarter to Month

### Tooltips
- Hover over any site name data point on the Page 2 risk chart to trigger the **Site Risk Profile** tooltip
- Hover over any cell in the Page 2 safety heatmap to trigger the **Safety Breakdown** tooltip

### Drill-through
Right-click any site name on Pages 1 or 2 to access site-level drill-through detail (if configured in your environment).

---

## Stakeholder Guide

| Stakeholder | Primary Page | Key Visuals | Key KPIs |
|---|---|---|---|
| CEO | Page 1 (Revenue toggle) | Waterfall, heatmap | Total Revenue, Sites Meeting All Targets, Production Gap Value |
| COO | Page 1 (Production toggle) | Area trend, scatter, heatmap | Capacity Utilization %, Total Downtime Hours |
| Operations Manager | Pages 1 & 2 | Scatter, risk column chart | Production Gap, Revenue at Risk, Composite Risk Score |
| Maintenance Manager | Page 2 | Downtime vs. cost scatter | Total Maintenance Cost, Unplanned Maintenance Ratio, Maint. Cost/Downtime Hr |
| HSE Manager | Page 2 | Safety heatmap + tooltip | Total Safety Incidents, Incident Free Sites, Safety Breakdown tooltip |
| Sustainability Lead | Page 3 | CO₂ trend, scatter, bar | CO₂ Gap, CO₂/barrel, Energy Intensity |
| Finance Director | Pages 1 & 3 | Waterfall, oil price vs. maint. line | Revenue at Risk, Production Gap Value USD, Maint. Cost/Downtime Hr |

---

## Tools & Technologies

| Tool | Usage |
|---|---|
| Power BI Desktop | Report development, DAX authoring, data modelling |
| DAX | All measures, calculated columns, dynamic insight text |
| Power Query (M) | Data transformation and table shaping |
| Field Parameters | Production / Revenue toggle on Page 1 |
| Bookmarks | Page state management for toggle navigation |
| Azure Maps / Bing Maps | Geographic site visualisation |
| Chart.js (documentation only) | Wireframe prototyping during design phase |

---

## Author

**Emmanuel Idowu**
Power BI Developer · Data Analytics Professional · Oil & Gas Operations Analytics

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Emmanuel_Idowu-0A66C2?style=flat&logo=linkedin)]([https://www.linkedin.com/in/emmanuelidowu](https://www.linkedin.com/in/emmanuel-idowu-analyst/ 
))

---

*Built as part of the ZoomCharts 4U Oil, Gas & Energy Operations Analytics Power BI Challenge — a structured portfolio project demonstrating executive-grade BI design for upstream and midstream energy operations.*
