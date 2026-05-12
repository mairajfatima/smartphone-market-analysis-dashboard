# 📱 Smartphone Market Analysis Dashboard

![Dashboard Preview](<img width="1071" height="596" alt="image" src="https://github.com/user-attachments/assets/038a6213-54d6-420b-8d17-abc762029010" />
# 📱 Smartphone Market Analysis Dashboard

> **Built in:** Microsoft Power BI Desktop  
> **Prepared by:** Mairaj Fatima  
> **Tool branding:** SmartInsight  
> **Dataset:** Smartphones Cleaned Dataset (CSV)  
> **Scope:** 980 smartphone models · 46 brands · Pricing in PKR (Pakistani Rupees)

---

## 📌 Project Overview

This end-to-end Power BI project analyzes the global smartphone market across dimensions of price, performance, connectivity, camera, battery, and consumer ratings. The goal is to help a business or consumer understand **where value lies** in a market that spans from PKR 3,940 (Lyf) to PKR 6,50,000 (Vertu) — a 162× price range.

The dashboard answers real business questions that a product manager, analyst, or smartphone buyer would ask: Which brand gives the best rating per PKR spent? Is 5G worth the premium? Which processor chip delivers the most performance? What is the real price of storage?

The project covers the full BI lifecycle:
1. Raw data ingestion
2. Power Query cleaning and transformation
3. Data modeling (star schema)
4. DAX measure development (82 measures)
5. Report design (7 pages, 6 content pages)
6. Storytelling and insight communication

---

## 🗂️ Dataset Description

**Source file:** `smartphones_cleaned_dataset.csv`  
**Rows:** 980 smartphone models  
**Brands:** 46 unique brands  

### Key columns in the raw dataset

| Column | Description |
|---|---|
| `brand_name` | Manufacturer name (samsung, apple, xiaomi etc.) |
| `model_name` | Full model name |
| `price` | Price in Pakistani Rupees (PKR) |
| `rating` | Consumer rating score (0–100 scale) |
| `has_5g` | Boolean — whether the phone supports 5G |
| `has_nfc` | Boolean — NFC support |
| `has_ir_blaster` | Boolean — infrared blaster |
| `processor_brand` | Chip maker (snapdragon, dimensity, bionic, kirin, etc.) |
| `num_cores` | Number of CPU cores |
| `processor_speed` | Clock speed in GHz |
| `battery_capacity` | Battery in mAh |
| `fast_charging_available` | Boolean — fast charging support |
| `fast_charging` | Fast charging wattage |
| `ram_capacity` | RAM in GB |
| `internal_memory` | Internal storage in GB |
| `screen_size` | Display size in inches |
| `refresh_rate` | Display refresh rate in Hz |
| `num_rear_cameras` | Number of rear cameras |
| `primary_camera_rear` | Main rear camera in megapixels |
| `primary_camera_front` | Selfie camera in megapixels |
| `extended_memory_available` | Boolean — expandable storage |
| `extended_upto` | Max expandable storage in GB |
| `os` | Operating system (Android, iOS, etc.) |
| `resolution_height` / `resolution_width` | Display resolution |
| `performance_score(%)` | Pre-computed composite hardware score (0–100) |
| `price_segment` | Categorical: Budget / Mid_Range / Premium |

---

## 🧹 Power Query — Data Cleaning & Transformation

All cleaning was done inside **Power Query Editor** before loading data into the model.

### Steps performed

**1. Source connection**  
Loaded the CSV file using `Get Data → Text/CSV`. Power Query auto-detected column types on import.

**2. Promoted headers**  
Used `Use First Row as Headers` to ensure the first row became column names, not data.

**3. Data type corrections**  
Manually corrected column types that Power Query misread:
- `price` → Whole Number (PKR values were sometimes read as text due to formatting)
- `has_5g`, `has_nfc`, `has_ir_blaster`, `fast_charging_available`, `extended_memory_available` → TRUE/FALSE (Boolean)
- `processor_speed` → Decimal Number
- `screen_size` → Decimal Number
- `primary_camera_rear`, `primary_camera_front` → Whole Number

**4. Handling null and missing values**  
- `fast_charging` wattage was null for phones where `fast_charging_available = FALSE` → replaced nulls with `0` using `Replace Values`
- `extended_upto` was null for phones without expandable storage → replaced with `0`
- `primary_camera_front` had some nulls for models with no selfie camera → replaced with `0`
- `processor_speed` had occasional nulls for entry-level chips → replaced with median value or `0`

**5. Text standardization**  
- `brand_name` and `processor_brand` were lowercased and trimmed using `Text.Lower()` and `Text.Trim()` to remove inconsistencies like " Samsung" vs "samsung"
- `os` column standardized — variants like "Android 12", "Android 13" were kept as-is since OS version analysis was not in scope

**6. Price segment column**  
The raw dataset had a `price_segment` column with values `Budget`, `Mid_Range`, `Premium`. These were validated:
- Budget = price < PKR 20,000 → 506 models
- Mid_Range = PKR 20,000–50,000 → 313 models
- Premium = above PKR 50,000 → 161 models

**7. Removing irrelevant columns**  
Any columns not needed for the analysis (internal IDs, scraping metadata, unnamed index columns) were removed using `Remove Columns`.

**8. Filtering outliers for analysis integrity**  
Vertu (PKR 6,50,000) and Royole (PKR 1,29,999) are real phones but extreme outliers. They were kept in the dataset (not removed) since they are legitimate data points — the Value for Money Score naturally handles them by scoring them near 0.

**9. Final load**  
After all transformations, the clean table was loaded as `Smartphones_cleaned_dataset` into the Power BI data model.

---

## 🗃️ Data Modeling

The data model uses a **star schema** with one central fact table and dimension tables.

```
Dim_Brand ──────────────────┐
Dim_Processor ──────────────┤
Dim_PriceSegment ───────────┼──── Smartphones_cleaned_dataset (Fact)
Dim_OS ─────────────────────┤
Dim_Connectivity ───────────┘
```

### Fact table
**`Smartphones_cleaned_dataset`** — the cleaned CSV loaded directly. Contains all numeric measures (price, rating, battery, RAM, storage, camera MP, performance score etc.) and foreign keys to dimension tables.

### Dimension tables
Created either by referencing the fact table or as separate small lookup tables:

| Table | Key column | Purpose |
|---|---|---|
| `Dim_Brand` | `brand_name` | Brand-level filtering and slicers |
| `Dim_Processor` | `processor_brand` | Processor slicer on Rating & Performance page |
| `Dim_PriceSegment` | `price_segment` | Segment slicer (Budget / Mid_Range / Premium) |
| `Dim_OS` | `os` | OS slicer on Pricing Analysis page |

### Relationships
All relationships are **one-to-many** from the dimension table to the fact table:
- `Dim_Brand[brand_name]` → `Smartphones_cleaned_dataset[brand_name]` (1:many)
- `Dim_Processor[processor_brand]` → `Smartphones_cleaned_dataset[processor_brand]` (1:many)
- `Dim_PriceSegment[price_segment]` → `Smartphones_cleaned_dataset[price_segment]` (1:many)

Cross-filter direction: **Single** (dimension filters fact, not reverse) to maintain standard star schema performance.

---

## 📐 DAX Measures (82 total)

All measures are organized in a dedicated **`_Measures`** table (a disconnected table used purely as a measure container — best practice to keep the model clean).

### Category 1 — Core KPIs

```dax
-- Total model count
Model Count = COUNTROWS(Smartphones_cleaned_dataset)

-- Total unique brands
Total Brands = DISTINCTCOUNT(Smartphones_cleaned_dataset[brand_name])

-- Average price across all models
Avg Price = AVERAGE(Smartphones_cleaned_dataset[price])

-- Average consumer rating
Avg Rating = AVERAGE(Smartphones_cleaned_dataset[rating])
-- Returns: 78/100 (overall), dynamic with slicers

-- Average performance score (composite hardware score 0–100%)
Avg Performance Score = AVERAGE(Smartphones_cleaned_dataset[performance_score(%)])
-- Returns: 68%

-- 5G adoption rate
5G % = DIVIDE(
    COUNTROWS(FILTER(Smartphones_cleaned_dataset, Smartphones_cleaned_dataset[has_5g] = TRUE())),
    [Model Count]
)
-- Returns: 56.02%

-- NFC adoption rate
NFC % = DIVIDE(
    COUNTROWS(FILTER(Smartphones_cleaned_dataset, Smartphones_cleaned_dataset[has_nfc] = TRUE())),
    [Model Count]
)
-- Returns: 40.1%

-- Fast charging adoption rate
Fast Charging % = DIVIDE(
    COUNTROWS(FILTER(Smartphones_cleaned_dataset, Smartphones_cleaned_dataset[fast_charging_available] = TRUE())),
    [Model Count]
)
-- Returns: 85.4%
```

### Category 2 — Pricing Measures

```dax
-- Average price of 5G phones only
Avg Price 5G = CALCULATE(
    [Avg Price],
    Smartphones_cleaned_dataset[has_5g] = TRUE()
)
-- Returns: PKR 43,200

-- Average price of Non-5G phones only
Avg Price Non-5G = CALCULATE(
    [Avg Price],
    Smartphones_cleaned_dataset[has_5g] = FALSE()
)
-- Returns: PKR 19,000

-- The premium you pay for 5G over Non-5G
5G Price Premium = [Avg Price 5G] - [Avg Price Non-5G]
-- Returns: PKR 24,200

-- Minimum price of 5G phones (used in dumbbell chart)
Min Price 5G = CALCULATE(
    MIN(Smartphones_cleaned_dataset[price]),
    Smartphones_cleaned_dataset[has_5g] = TRUE()
)

-- Maximum price of 5G phones (used in dumbbell chart)
Max Price 5G = CALCULATE(
    MAX(Smartphones_cleaned_dataset[price]),
    Smartphones_cleaned_dataset[has_5g] = TRUE()
)

-- Min / Max price of Non-5G phones (used in dumbbell chart)
Min Price Non-5G = CALCULATE(MIN(Smartphones_cleaned_dataset[price]), Smartphones_cleaned_dataset[has_5g] = FALSE())
Max Price Non-5G = CALCULATE(MAX(Smartphones_cleaned_dataset[price]), Smartphones_cleaned_dataset[has_5g] = FALSE())
```

### Category 3 — Rating & Performance

```dax
-- Average rating filtered by processor brand slicer context
Avg Rating by Processor = CALCULATE(
    [Avg Rating],
    ALLEXCEPT(Smartphones_cleaned_dataset, Smartphones_cleaned_dataset[processor_brand])
)

-- Top rated brand name (text measure)
Top Rated Brand = 
VAR TopBrand = TOPN(1, ALL(Smartphones_cleaned_dataset[brand_name]), [Avg Rating], DESC)
RETURN CONCATENATEX(TopBrand, Smartphones_cleaned_dataset[brand_name])
-- Returns: "leitz" dynamically (changes with slicers)

-- Average CPU core count
Avg Cores = AVERAGE(Smartphones_cleaned_dataset[num_cores])
-- Returns: 7.77

-- Average processor clock speed
Avg Processor Speed = AVERAGE(Smartphones_cleaned_dataset[processor_speed])
-- Returns: 2.43 GHz
```

### Category 4 — Value for Money Score (most complex measure)

```dax
-- Step 1: Raw value ratio per phone (rating ÷ price)
Value Score Raw = DIVIDE([Avg Rating], [Avg Price])

-- Step 2: Find the global minimum and maximum ratios
VAR MinRatio = MINX(ALL(Smartphones_cleaned_dataset), DIVIDE(Smartphones_cleaned_dataset[rating], Smartphones_cleaned_dataset[price]))
VAR MaxRatio = MAXX(ALL(Smartphones_cleaned_dataset), DIVIDE(Smartphones_cleaned_dataset[rating], Smartphones_cleaned_dataset[price]))

-- Step 3: Normalize to 0–100 scale (min-max normalization)
Value for Money Score = 
VAR CurrentRatio = [Value Score Raw]
VAR MinR = MINX(ALL(Smartphones_cleaned_dataset), DIVIDE(Smartphones_cleaned_dataset[rating], Smartphones_cleaned_dataset[price]))
VAR MaxR = MAXX(ALL(Smartphones_cleaned_dataset), DIVIDE(Smartphones_cleaned_dataset[rating], Smartphones_cleaned_dataset[price]))
RETURN DIVIDE(CurrentRatio - MinR, MaxR - MinR) * 100
-- Returns: 21/100 (overall average)
-- Lyf scores ~87/100 (best value), Vertu scores ~0/100 (worst value)
```

### Category 5 — Hardware & Feature Measures

```dax
Avg Battery       = AVERAGE(Smartphones_cleaned_dataset[battery_capacity])   -- 4,764 mAh
Avg RAM           = AVERAGE(Smartphones_cleaned_dataset[ram_capacity])        -- 6.56 GB
Avg Storage       = AVERAGE(Smartphones_cleaned_dataset[internal_memory])     -- 141 GB
Avg Screen Size   = AVERAGE(Smartphones_cleaned_dataset[screen_size])
Avg Rear Camera MP = AVERAGE(Smartphones_cleaned_dataset[primary_camera_rear])
Avg Front Camera MP = AVERAGE(Smartphones_cleaned_dataset[primary_camera_front])

-- Feature Adoption Score: composite of 5G + fast charging + NFC rates
Feature Adoption Score = ([5G %] + [Fast Charging %] + [NFC %]) / 3

-- Camera efficiency: megapixels per PKR
Camera Price Efficiency = DIVIDE([Avg Rear Camera MP], [Avg Price])
```

---

## ❓ Business Questions Answered

| # | Business Question | Page | Key Finding |
|---|---|---|---|
| 1 | What is the overall state of the smartphone market? | Home Page | 980 models, 46 brands, avg PKR 33K, 56% 5G, avg rating 78/100 |
| 2 | Which brands are most loved by consumers? | Home & Rating & Performance | Leitz, Lenovo, Sharp lead by avg rating — but are niche. iQOO, OnePlus lead at scale |
| 3 | How much more does a 5G phone cost? | Pricing Analysis | PKR 24,200 premium on average — nearly the full cost of a budget phone |
| 4 | Which processor chip delivers the best performance? | Rating & Performance | Kirin (84%) → Google Tensor (79%) → Snapdragon (76%) → Dimensity (74%) |
| 5 | How does storage affect price? | Rating & Performance | 128GB → 256GB doubles price from PKR 28K to PKR 61K. 512GB costs PKR 1.22L on average |
| 6 | Which brands offer 5G across their lineup vs selectively? | Features & Connectivity | Asus, Royole, ZTE, Leitz sell ONLY 5G models. Samsung/Apple/Xiaomi cover both segments |
| 7 | Which brands deliver the best value for money? | Brand Intelligence | Lyf (87/100 value score), itel (67), Gionee (54). Apple (4.4), Vertu (0) at bottom |
| 8 | Does a higher price always mean a better rating? | Rating & Performance | No. Apple avg rating 76.9 vs OnePlus 81.9 at less than half the price |
| 9 | What is the most common smartphone configuration? | Pricing Analysis (treemap) | 128GB storage, 6–8GB RAM — by far the dominant hardware configuration |
| 10 | Is 5G worth it in the Budget segment? | Features & Connectivity | Budget 5G adoption is only 27% vs 90% in Premium — 5G in Budget is rare and costs more |

---

## 📊 Report Pages — Full Breakdown

### Page 1 — Home Page
**Purpose:** Executive overview. A landing page that gives any viewer instant context before they explore deeper.

**Visuals:**
- 5 KPI cards: Avg Rating (78/100) · Avg Performance Score (68%) · Value for Money Score (21/100, highlighted) · Avg Price (PKR 33K) · Models with 5G (56.02%)
- Bar chart: Top Brands by Avg Rating (Leitz, Lenovo, Sharp, Royole, Doogee, Asus, ZTE)
- Bar chart: Market Affordability — avg price by selected brands showing the price landscape

**Navigation:** Left sidebar with icon buttons linking to all 6 pages.

---

### Page 2 — Market Overview
**Purpose:** Distribution of models across brands, segments, and OS — the "who is in this market" page.

---

### Page 3 — Pricing Analysis
**Purpose:** Deep-dive into price structure, 5G premium, brand positioning by price, hardware cost.

**Visuals:**
- 4 KPI cards: Avg Price (PKR 33K) · Avg Price 5G (PKR 43.2K) · Avg Price Non-5G (PKR 19K) · 5G Price Premium (PKR 24K)
- Horizontal bar: Avg Price by Brand (full brand list, scrollable)
- Treemap: Hardware Distribution — RAM × Storage combinations sized by model count
- Dumbbell chart: 5G vs Non-5G Price Range by Brand — price span from cheapest non-5G to most expensive 5G per brand

**Slicers:** Price Segment · Brand Name · Operating System

---

### Page 4 — Rating & Performance
**Purpose:** Which brands and processors make phones people actually love? Does price buy better ratings?

**Visuals:**
- 4 KPI cards: Avg Rating by Processor (78.26) · Top Rated Brand (Leitz, dynamic) · Avg Cores (7.77) · Avg Performance Score (68%)
- Horizontal bar: Avg Rating by Brand (top N, sorted descending)
- Scatter plot: Rating vs Price — each dot = one brand, quadrant analysis reveals value vs overpriced positioning
- Area chart: Avg Price by Internal Memory — shows exponential price growth from 32GB to 1TB
- Horizontal bar: Avg Performance Score by Processor Brand — Kirin, Google, Snapdragon, Dimensity, Exynos, Bionic, Helio

**Slicers:** Price Segment · Brand Name · Processor Brand · Model Name

---

### Page 5 — Features & Connectivity
**Purpose:** 5G, NFC, fast charging, IR blaster — who has what and what does it cost?

---

### Page 6 — Specifications
**Purpose:** Camera, battery, display, RAM, storage deep-dive across the market.

---

### Page 7 — Brand Intelligence
**Purpose:** Value for Money Score ranking. The final verdict page — which brands actually deserve your money.

---

## 🎨 Design Decisions

| Decision | Reason |
|---|---|
| Dark navy + teal color palette | Professional, tech-forward feel — consistent with SmartInsight branding |
| Left sidebar navigation | Power BI canvas navigation using bookmark + button actions for a app-like experience |
| Alternating dark/teal bar colors | Improves readability of long brand lists without gridlines |
| Dumbbell chart for 5G pricing | Best visual for showing a range with two endpoints — more informative than two separate bars |
| Treemap for hardware distribution | Shows proportion + combination simultaneously — better than a matrix for this data |
| Scatter plot for Rating vs Price | Only chart type that can plot two continuous measures against each other with brand labels as dots |
| Value for Money Score highlighted KPI | White outline border on this card signals it is the primary insight of the Home Page |

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
| Total DAX measures | 82 |
| Report pages | 7 (1 home + 6 content) |

---

## 🛠️ Tools & Technologies

| Tool | Usage |
|---|---|
| Microsoft Power BI Desktop | Report building, DAX, data modeling |
| Power Query (M language) | Data cleaning and transformation |
| DAX (Data Analysis Expressions) | All 82 business measures |
| CSV (source data) | smartphones_cleaned_dataset.csv |
| Star schema | Data model architecture |

---

## 📁 Files in This Repository

```
📦 smartphone-market-analysis-dashboard/
├── 📊 Smartphone_Market_Analysis.pbix      ← Main Power BI report file
├── 📄 smartphones_cleaned_dataset.csv      ← Source dataset
├── 📄 smartphones_Market Analysis Dashbaord.pdf   ← PDF
└── 📝 README.md                            ← This file
```

---

## 🚀 How to Open This Project

1. Download and install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free)
2. Clone or download this repository
3. Open `Smartphone_Market_Analysis.pbix` in Power BI Desktop
4. If prompted about data source path, update the CSV path to your local `smartphones_cleaned_dataset.csv` location via **Transform Data → Data Source Settings**
5. Click **Refresh** to reload data

---

## 👩‍💻 Author

**Mairaj Fatima**  
Power BI Developer & Data Analyst  
Dashboard branding: SmartInsight  

---

*This project was built as a complete end-to-end Power BI portfolio piece covering data cleaning, modeling, DAX development, and visual storytelling across the global smartphone market.*
)
