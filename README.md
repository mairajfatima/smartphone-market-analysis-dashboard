# 📱 Smartphone Market Analysis Dashboard

![Dashboard Preview](<img width="1071" height="596" alt="image" src="https://github.com/user-attachments/assets/038a6213-54d6-420b-8d17-abc762029010" />

# Smartphone Market Analysis Dashboard

**Built in:** Microsoft Power BI Desktop  
**Prepared by:** Mairaj Fatima  
**Tool branding:** SmartInsight  
**Dataset:** Smartphones Cleaned Dataset (CSV)  
**Scope:** 980 smartphone models · 46 brands · Pricing in PKR (Pakistani Rupees)

---

## 📌 Project Overview

This end-to-end Power BI project analyzes the global smartphone market across dimensions of price, performance, connectivity, camera, battery, and consumer ratings. The goal is to help a business or consumer understand where value lies in a market that spans from PKR 3,940 (Lyf) to PKR 6,50,000 (Vertu) — a 162× price range.

The dashboard answers real business questions that a product manager, analyst, or smartphone buyer would ask: Which brand gives the best rating per PKR spent? Is 5G worth the premium? Which processor chip delivers the most performance? What is the real price of storage? How does a specific model compare against its direct price-segment competitors?

The project covers the full BI lifecycle:

- Raw data ingestion
- Power Query cleaning and transformation
- Data modeling (star schema)
- DAX measure development (90+ measures across 8 pages)
- Report design (8 pages — 1 home + 7 content pages)
- Storytelling and insight communication

---

## 🗂️ Dataset Description

**Source file:** `smartphones_cleaned_dataset.csv`  
**Rows:** 980 smartphone models  
**Brands:** 46 unique brands

### Key columns in the raw dataset

| Column | Description |
|---|---|
| brand_name | Manufacturer name (samsung, apple, xiaomi etc.) |
| model_name | Full model name |
| price | Price in Pakistani Rupees (PKR) |
| rating | Consumer rating score (0–100 scale) |
| has_5g | Boolean — whether the phone supports 5G |
| has_nfc | Boolean — NFC support |
| has_ir_blaster | Boolean — infrared blaster |
| fast_charging_available | Boolean — fast charging support |
| fast_charging | Fast charging wattage in watts |
| processor_brand | Chip maker (snapdragon, dimensity, bionic, kirin, etc.) |
| num_cores | Number of CPU cores |
| processor_speed | Clock speed in GHz |
| battery_capacity | Battery in mAh |
| ram_capacity | RAM in GB |
| internal_memory | Internal storage in GB |
| screen_size | Display size in inches |
| refresh_rate | Display refresh rate in Hz |
| num_rear_cameras | Number of rear cameras |
| primary_camera_rear | Main rear camera in megapixels |
| primary_camera_front | Selfie camera in megapixels |
| extended_memory_available | Boolean — expandable storage |
| extended_upto | Max expandable storage in GB |
| os | Operating system (Android, iOS, etc.) |
| resolution_height / resolution_width | Display resolution |
| performance_score (%) | Pre-computed composite hardware score (0–100) |
| price_segment | Categorical: Budget / Mid_Range / Premium |

---

## 🧹 Power Query — Data Cleaning & Transformation

All cleaning was done inside Power Query Editor before loading data into the model.

### Steps performed

**1. Source connection**  
Loaded the CSV file using Get Data → Text/CSV. Power Query auto-detected column types on import.

**2. Promoted headers**  
Used Use First Row as Headers to ensure the first row became column names, not data.

**3. Data type corrections**  
Manually corrected column types that Power Query misread:
- `price` → Whole Number
- `has_5g`, `has_nfc`, `has_ir_blaster`, `fast_charging_available`, `extended_memory_available` → TRUE/FALSE (Boolean)
- `processor_speed` → Decimal Number
- `screen_size` → Decimal Number
- `primary_camera_rear`, `primary_camera_front` → Whole Number

**4. Handling null and missing values**
- `fast_charging` wattage was null for phones where `fast_charging_available = FALSE` → replaced nulls with 0
- `extended_upto` was null for phones without expandable storage → replaced with 0
- `primary_camera_front` had some nulls for models with no selfie camera → replaced with 0
- `processor_speed` had occasional nulls for entry-level chips → replaced with 0

**5. Text standardization**  
`brand_name` and `processor_brand` were lowercased and trimmed using `Text.Lower()` and `Text.Trim()` to remove inconsistencies like " Samsung" vs "samsung".

**6. Price segment validation**  
The raw dataset had a `price_segment` column validated as:
- Budget = price < PKR 20,000 → 506 models
- Mid_Range = PKR 20,000–50,000 → 313 models
- Premium = above PKR 50,000 → 161 models

**7. Removing irrelevant columns**  
Any columns not needed for the analysis (internal IDs, scraping metadata, unnamed index columns) were removed.

**8. Filtering outliers for analysis integrity**  
Vertu (PKR 6,50,000) and Royole (PKR 1,29,999) are real phones but extreme outliers. They were kept in the dataset since they are legitimate data points — the Value for Money Score naturally handles them by scoring them near 0.

**9. Final load**  
After all transformations, the clean table was loaded as `Smartphones_cleaned_dataset` into the Power BI data model.

---

## 🗃️ Data Modeling

The data model uses a **star schema** with one central fact table and four dimension tables.

```
Dim_Brand ──────────────────┐
Dim_Processor ──────────────┤
Dim_PriceSegment ───────────┼──── Smartphones_cleaned_dataset (Fact)
Dim_OS ─────────────────────┘
```

### Fact table
`Smartphones_cleaned_dataset` — the cleaned CSV loaded directly. Contains all numeric measures (price, rating, battery, RAM, storage, camera MP, performance score etc.) and foreign keys to dimension tables.

### Dimension tables

| Table | Key column | Purpose |
|---|---|---|
| Dim_Brand | brand_name | Brand-level filtering and slicers |
| Dim_Processor | processor_brand | Processor slicer on Rating & Performance page |
| Dim_PriceSegment | price_segment | Segment slicer (Budget / Mid_Range / Premium) |
| Dim_OS | os | OS slicer on Pricing Analysis page |

### Relationships
All relationships are **one-to-many** from the dimension table to the fact table:
- `Dim_Brand[brand_name]` → `Smartphones_cleaned_dataset[brand_name]` (1:many)
- `Dim_Processor[processor_brand]` → `Smartphones_cleaned_dataset[processor_brand]` (1:many)
- `Dim_PriceSegment[price_segment]` → `Smartphones_cleaned_dataset[price_segment]` (1:many)
- `Dim_OS[os]` → `Smartphones_cleaned_dataset[os]` (1:many)

**Cross-filter direction:** Single (dimension filters fact, not reverse) — standard star schema best practice.

### Additional disconnected table — Model Intelligence
`Spec_Labels` — a calculated DAX table with 7 rows (RAM, Storage, Battery, Refresh Rate, Performance, Fast Charging, Camera). Used as the category axis for the Spec Strength diverging bar chart on the Model Intelligence page. It has no relationship to the fact table — it is driven entirely by SWITCH measures.

---

## 📐 DAX Measures

All measures are organized in a dedicated **Measures Table** (a disconnected table used purely as a measure container — best practice to keep the model clean). Measures are grouped into display folders by theme.

### Display folders
- Core KPIs
- Pricing
- Rating & Performance
- Features
- Battery
- Camera
- Memory
- Value Scores
- 5G Price Comparison
- Feature Adoption
- **Model Intelligence** ← new folder added for page 8
- **Model Intelligence\Spec Matrix** ← 14 spec comparison measures
- **Model Intelligence\Radar** ← 8 normalized gap measures

### Category 1 — Core KPIs

```dax
Model Count = COUNTROWS(Smartphones_cleaned_dataset)

Total Brands = DISTINCTCOUNT(Smartphones_cleaned_dataset[brand_name])

Avg Price = AVERAGE(Smartphones_cleaned_dataset[price])

Avg Rating = AVERAGE(Smartphones_cleaned_dataset[rating])

Avg Performance Score = AVERAGE(Smartphones_cleaned_dataset[performance_score (%)])

5G % = DIVIDE(
    COUNTROWS(FILTER(Smartphones_cleaned_dataset, Smartphones_cleaned_dataset[has_5g] = TRUE())),
    [Model Count]
)
```

### Category 2 — Pricing Measures

```dax
Avg Price 5G = ROUND(
    CALCULATE(AVERAGE(Smartphones_cleaned_dataset[price]), Smartphones_cleaned_dataset[has_5g] = TRUE()),
    0
)

Avg Price Non-5G = CALCULATE(
    AVERAGE(Smartphones_cleaned_dataset[price]),
    FILTER(Smartphones_cleaned_dataset, Smartphones_cleaned_dataset[has_5g] = FALSE())
)

5G Price Premium = [Avg Price 5G] - [Avg Price Non-5G]
```

### Category 3 — Value for Money Score (most complex measure)

```dax
Value Score Raw =
    AVERAGEX(
        Smartphones_cleaned_dataset,
        Smartphones_cleaned_dataset[rating] / Smartphones_cleaned_dataset[price]
    )

Value for Money Score =
VAR MinVal = MINX(ALL(Smartphones_cleaned_dataset), Smartphones_cleaned_dataset[rating] / Smartphones_cleaned_dataset[price])
VAR MaxVal = MAXX(ALL(Smartphones_cleaned_dataset), Smartphones_cleaned_dataset[rating] / Smartphones_cleaned_dataset[price])
RETURN
    DIVIDE([Value Score Raw] - MinVal, MaxVal - MinVal)
```

> **Note:** `Value Score Raw` uses `AVERAGEX` (average of per-row ratios) — not `DIVIDE(AVG(rating), AVG(price))`. This is the statistically correct approach. The two formulas produce different results and are not interchangeable.

### Category 4 — Model Intelligence Measures (new — Page 8)

These measures are designed exclusively for the Model Intelligence page. They all use a consistent pattern: extract the selected model using `ALLSELECTED` to avoid context collision when slicers are active simultaneously.

```dax
-- Core pattern used by all Model Intelligence measures
VAR SelectedModel =
    CALCULATE(
        SELECTEDVALUE(Smartphones_cleaned_dataset[model]),
        ALLSELECTED(Smartphones_cleaned_dataset[model])
    )
```

**KPI measures:**

```dax
Price Gap vs Segment =
VAR SelectedModel = CALCULATE(SELECTEDVALUE(Smartphones_cleaned_dataset[model]), ALLSELECTED(Smartphones_cleaned_dataset[model]))
VAR SelectedModelPrice = CALCULATE(MIN(Smartphones_cleaned_dataset[price]), ALL(Smartphones_cleaned_dataset), Smartphones_cleaned_dataset[model] = SelectedModel)
VAR SelectedSegment = CALCULATE(SELECTEDVALUE(Smartphones_cleaned_dataset[price_segment]), ALL(Smartphones_cleaned_dataset), Smartphones_cleaned_dataset[model] = SelectedModel)
VAR SegmentAvgPrice = CALCULATE(AVERAGE(Smartphones_cleaned_dataset[price]), ALL(Smartphones_cleaned_dataset), Smartphones_cleaned_dataset[price_segment] = SelectedSegment)
RETURN IF(ISBLANK(SelectedModel) || ISBLANK(SelectedModelPrice), BLANK(), SelectedModelPrice - SegmentAvgPrice)
-- Positive = model is overpriced vs segment. Negative = model is underpriced vs segment.

Performance Percentile =
-- Ranks selected model's performance score across entire dataset
-- Returns 0-100. Higher = better. Rank 1 → ~100th percentile.
-- Formula: (1 - DIVIDE(ModelRank, TotalModels)) * 100

VFM Index =
-- (Rating × Performance Score) / Price × 100,000
-- Higher = better value disruptor. Used for competitor ranking chart.
-- Separate from Value for Money Score which stays on other pages.

Battery Efficiency Score =
-- battery_capacity / (price / 1000)
-- Returns mAh per PKR 1,000 spent. Higher = better battery value.

Rating Gap vs Segment =
-- Selected model rating minus average rating of all models in same segment
-- Format: +4.2 (above avg) or -3.1 (below avg)

Feature Score =
-- Count of TRUE out of 5 binary features: 5G + NFC + IR Blaster + Fast Charging + Extended Memory
-- Returns 0–5. Returns BLANK if no model selected.
```

**Spec Matrix measures (14 total — Selected vs Segment Avg):**

For each of 7 specs (RAM, Storage, Battery, Refresh Rate, Performance, Fast Charging, Camera), two measures exist:
- `Selected [Spec]` — returns the exact value for the selected model
- `Segment Avg [Spec]` — returns the average value across all models in the same price segment

**Radar/Diverging bar measures (8 total):**

```dax
-- Pattern for all 7 normalized gap measures
RAM vs Segment % = DIVIDE([Selected RAM] - [Segment Avg RAM], [Segment Avg RAM]) * 100
-- Returns percentage above/below segment average. 0 = at average. +50 = 50% better.

-- SWITCH measure for the diverging bar chart
Spec vs Segment Avg % =
SWITCH(
    SELECTEDVALUE(Spec_Labels[Spec]),
    "RAM", [RAM vs Segment %],
    "Storage", [Storage vs Segment %],
    "Battery", [Battery vs Segment %],
    "Refresh Rate", [Refresh Rate vs Segment %],
    "Performance", [Performance vs Segment %],
    "Fast Charging", [Fast Charging vs Segment %],
    "Camera", [Camera vs Segment %],
    BLANK()
)
-- Used with Spec_Labels[Spec] as Y-axis category on the diverging bar chart
```

---

## ❓ Business Questions Answered

| # | Business Question | Page | Key Finding |
|---|---|---|---|
| 1 | What is the overall state of the smartphone market? | Home Page | 980 models, 46 brands, avg PKR 33K, 56% 5G, avg rating 78/100 |
| 2 | Which brands are most loved by consumers? | Rating & Performance | Leitz, Lenovo, Sharp lead by avg rating — iQOO, OnePlus lead at scale |
| 3 | How much more does a 5G phone cost? | Pricing Analysis | PKR 24,200 premium on average — nearly the full cost of a budget phone |
| 4 | Which processor chip delivers the best performance? | Rating & Performance | Kirin (84%) → Google Tensor (79%) → Snapdragon (76%) → Dimensity (74%) |
| 5 | How does storage affect price? | Pricing Analysis | 128GB → 256GB doubles price from PKR 28K to PKR 61K |
| 6 | Which brands offer 5G across their full lineup? | Features & Connectivity | Asus, Royole, ZTE, Leitz sell ONLY 5G models |
| 7 | Which brands deliver the best value for money? | Brand Intelligence | Lyf (87/100), itel (67), Gionee (54). Apple (4.4), Vertu (0) at bottom |
| 8 | Does a higher price always mean a better rating? | Rating & Performance | No. Apple avg 76.9 vs OnePlus 81.9 at less than half the price |
| 9 | What is the most common smartphone configuration? | Pricing Analysis | 128GB storage, 6–8GB RAM — dominant hardware configuration |
| 10 | Is 5G worth it in the Budget segment? | Features & Connectivity | Budget 5G adoption only 27% vs 90% Premium — rare and costs more |
| 11 | Is this specific model overpriced for its segment? | Model Intelligence | Price Gap vs Segment KPI — positive = overpriced, negative = underpriced |
| 12 | How does this model's hardware compare to segment peers? | Model Intelligence | Spec Strength diverging bar — each bar shows % above/below segment average |
| 13 | What drives this model's value score? | Model Intelligence | Decomposition Tree drills brand → processor → model to root-cause VFM score | What drives this model's value score? | Model Intelligence | Decomposition Tree drills brand → processor → model to root-cause VFM score |

---

## 📊 Report Pages — Full Breakdown

### Page 1 — Home Page
**Purpose:** Executive overview. Landing page giving instant context before deeper exploration.

**Visuals:**
- 5 KPI cards: Avg Rating · Avg Performance Score · Value for Money Score · Avg Price · 5G adoption %
- Bar chart: Top Brands by Avg Rating
- Bar chart: Market Affordability — avg price by brand
- Left sidebar navigation with bookmark buttons linking to all pages

---

### Page 2 — Market Overview
**Purpose:** Distribution of models across brands, segments, and OS — the "who is in this market" page.

---

### Page 3 — Pricing Analysis
**Purpose:** Deep-dive into price structure, 5G premium, brand positioning by price, hardware cost.

**Visuals:**
- 4 KPI cards: Avg Price · Avg Price 5G · Avg Price Non-5G · 5G Price Premium
- Horizontal bar: Avg Price by Brand
- Treemap: Hardware Distribution — RAM × Storage combinations sized by model count
- Dumbbell chart: 5G vs Non-5G Price Range by Brand

---

### Page 4 — Rating & Performance
**Purpose:** Which brands and processors make phones people actually love?

**Visuals:**
- 4 KPI cards: Avg Rating by Processor · Top Rated Brand · Avg Cores · Avg Performance Score
- Horizontal bar: Avg Rating by Brand
- Scatter plot: Rating vs Price by brand (quadrant analysis)
- Area chart: Avg Price by Internal Memory
- Horizontal bar: Avg Performance Score by Processor Brand

---

### Page 5 — Features & Connectivity
**Purpose:** 5G, NFC, fast charging, IR blaster adoption across brands and segments.

---

### Page 6 — Specifications
**Purpose:** Camera, battery, display, RAM, storage deep-dive across the market.

---

### Page 7 — Brand Intelligence
**Purpose:** Value for Money Score ranking — the final verdict on which brands deserve your money.

---

### Page 8 — Model Intelligence *(new)*
**Purpose:** Single-model competitive benchmarking engine. The user selects one specific smartphone model via slicer and the entire page isolates that model's performance against its direct price-segment rivals. Answers the question: "Is this phone a market leader, fair value, or overpriced?"

**Slicers:**
- Price Segment (multi-select)
- Brand Name (single select recommended)
- Operating System (multi-select)
- Model Name (**single select — mandatory** for all KPIs and charts to activate)

**Visuals:**

**Row 1 — KPI Cards (4 cards)**

| Card | Measure | Interpretation |
|---|---|---|
| Price Gap vs Segment | Selected price minus segment average | Negative = underpriced (good). Positive = overpriced. |
| Performance Percentile | Rank vs full dataset (0–100%) | 98% = top 2% performer across all 980 models |
| VFM Index | Rating × Performance / Price × 100,000 | Higher = better value for money on this page |
| Battery Efficiency Score | mAh per PKR 1,000 spent | Higher = more battery per rupee |

**Row 2 — Two charts side by side**

- **VFM Index by Model** (horizontal bar chart) — shows selected model's VFM Index score. Responds to Model Name slicer.
- **Spec Strength vs Segment Average** (diverging bar chart) — 7 horizontal bars (RAM, Storage, Battery, Refresh Rate, Performance, Fast Charging, Camera). Each bar shows percentage above or below segment average. Bars going right = beating peers. Bars going left = underdelivering. Powered by `Spec_Labels` disconnected table + `Spec vs Segment Avg %` SWITCH measure.

**Row 3 — Decomposition Tree (full width)**
- Analyze: `VFM Index`
- Explain By: `brand_name` → `processor_brand` → `model`
- Purpose: Root-cause why a model scores high or low on value

**Row 4 — Brand Model Catalogue (independent section)**
- Text instruction: "Select a brand from Brand Name slicer to explore all its models. Select a model from Model Name slicer to activate KPI cards and charts above."
- Table visual: Model · Price Rs · Rating · RAM GB · Battery mAh · Refresh Rate Hz · Camera MP · Performance Score
- Interaction: Responds only to Brand Name slicer. Isolated from Model Name slicer via Edit Interactions.

**Key design decisions for this page:**
- All KPI measures use `ALLSELECTED` pattern to correctly extract the selected model's values even when multiple slicers are active simultaneously
- `SELECTEDVALUE` alone is not used — it returns BLANK when multiple slicer values exist. The `ALLSELECTED` wrapper isolates the model slicer context correctly
- Spec comparison uses percentage normalization (% above/below segment average) — not raw values — so specs with different scales (battery mAh vs RAM GB) can coexist on the same chart axis without distortion
- Page intentionally supports only single-model selection. Multi-model comparison is handled on Specifications and Brand Intelligence pages

---

## 🔢 Key Numbers to Remember

| Metric | Value |
|---|---|
| Total models analyzed | 980 |
| Total brands | 46 |
| Price range | PKR 3,940 (Lyf) → PKR 6,50,000 (Vertu) |
| Avg price | PKR 32,521 |
| Avg rating | 78 / 100 |
| Avg performance score | 68% |
| 5G adoption | 56.02% overall · 90% Premium · 27% Budget |
| NFC adoption | 40.1% |
| Fast charging adoption | 85.4% |
| 5G price premium | PKR 24,200 |
| Avg battery | 4,764 mAh |
| Avg RAM | 6.56 GB |
| Avg storage | 141 GB |
| Most common storage tier | 128 GB (523 of 980 models) |
| Top rated brand | Leitz (89/100 avg rating, 1 model) |
| Best value brand | Lyf (87/100 value score) |
| Worst value brand | Vertu (0/100 value score) |
| Total DAX measures | 90+ (82 core + 8 Model Intelligence pages) |
| Report pages | 8 (1 home + 7 content) |

---

## 🎨 Design Decisions

| Decision | Reason |
|---|---|
| Dark navy + teal color palette | Professional, tech-forward — consistent with SmartInsight branding |
| Left sidebar navigation | Bookmark + button actions for app-like experience |
| Alternating dark/teal bar colors | Readability of long brand lists without gridlines |
| Dumbbell chart for 5G pricing | Shows range with two endpoints — more informative than two bars |
| Treemap for hardware distribution | Shows proportion + combination simultaneously |
| Scatter plot for Rating vs Price | Only chart that plots two continuous measures with brand labels |
| Value for Money Score highlighted KPI | White outline border signals primary insight of Home Page |
| Diverging bar for spec comparison | Normalizes different-scale specs to percentage — avoids axis distortion |
| ALLSELECTED pattern for Model Intelligence | Prevents filter context collision when multiple slicers active simultaneously |
| Single-select Model Name slicer | SELECTEDVALUE requires exactly one selection — enforced at slicer level |
| Disconnected Spec_Labels table | Enables category-driven SWITCH measure for diverging bar — no relationship to fact table needed |

---

## 🛠️ Tools & Technologies

| Tool | Usage |
|---|---|
| Microsoft Power BI Desktop | Report building, DAX, data modeling |
| Power Query (M language) | Data cleaning and transformation |
| DAX (Data Analysis Expressions) | All 90+ business measures |
| CSV (source data) | smartphones_cleaned_dataset.csv |
| Star schema | Data model architecture |
| Power BI MCP (Model Context Protocol) | Backend measure creation and management via AI assistant |

---

## 📁 Files in This Repository

```
📦 smartphone-market-analysis-dashboard/
├── 📊 Smartphone_Market_Analysis.pbix      ← Main Power BI report file
├── 📄 smartphones_cleaned_dataset.csv      ← Source dataset
├── 📄 Smartphone_Market_Analysis.pdf       ← PDF export
└── 📝 README.md                            ← This file
```

---

## 🚀 How to Open This Project

1. Download and install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free)
2. Clone or download this repository
3. Open `Smartphone_Market_Analysis.pbix` in Power BI Desktop
4. If prompted about data source path, update the CSV path via Transform Data → Data Source Settings
5. Click Refresh to reload data
6. On the Model Intelligence page, select a single brand and single model from the slicers to activate all KPI cards and charts

---

## 👩‍💻 Author

**Mairaj Fatima**  
Power BI Developer & Data Analyst  
Dashboard branding: SmartInsight

