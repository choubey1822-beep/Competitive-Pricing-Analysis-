# 🚌 Bus Price Benchmarking System
### Automated competitive pricing analysis for bus operators

[![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python)](https://python.org)
[![Colab](https://img.shields.io/badge/Open%20in-Colab-orange?logo=googlecolab)](YOUR_COLAB_LINK_HERE)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## 📌 Project Overview

This project builds a **price benchmarking and flagging system** for a bus operator.  
Given a dataset of bus listings (price, load, ratings, route, etc.), the system:

1. **Identifies similar buses** — groups buses by Route × Bus Type × Rating Tier
2. **Benchmarks prices** — calculates the peer group median as a fair market price
3. **Flags pricing issues** — flags buses priced 10%+ or 20%+ away from the benchmark
4. **Quantifies the gap** — shows the exact ₹ and % difference for every route
5. **Proposes automation** — describes a 5-stage MVP pipeline from manual to fully automated

> Built as a take-home assignment for a **Data & Growth Intern** role.  
> Tools used: Python · pandas · matplotlib · seaborn · openpyxl · Google Colab

---

## 📊 Key Results

| Metric | Result |
|---|---|
| Total buses analysed | 500 |
| Peer groups created | 60 (avg 8.3 buses per group) |
| OurBus routes analysed | 105 |
| Routes flagged for action | **73 (69.5%)** |
| Most overpriced route | Delhi-Agra Luxury Tier C → **+54.6%** above median |
| Most underpriced route | Jaipur-Delhi AC Sleeper Tier B → **-58.5%** below median |
| Estimated daily revenue loss | ₹11,620/day on the most underpriced route alone |
| Overall avg OurBus variance | 0.8% (broadly fair, but individual routes vary widely) |

---

## 🗂️ Repository Structure

```
bus-price-benchmarking-system/
│
├── README.md
├── notebook/
│   └── Bus_Price_Benchmarking.ipynb   ← Full pipeline (11 cells)
├── excel/
│   └── Bus_Price_Benchmarking_Assignment.xlsx  ← Primary deliverable
├── docs/
│   └── Bus_Price_Benchmarking_Project_Report.docx
├── outputs/
│   ├── bus_price_flagging_output.csv  ← All 500 buses with flags
│   ├── our_bus_flags_only.csv         ← OurBus rows only
│   ├── action_required_flags.csv      ← 73 flagged routes
│   ├── eda_charts.png
│   ├── flagging_analysis.png
│   └── deep_dive_charts.png
```

---

## 🔬 Methodology

### Step 1 — Define Similar Buses

Buses are grouped using a **3-factor Composite Similarity Key**:

```
similarity_key = Route + "__" + Bus Type + "__" + Rating Tier
```

Example: `Delhi-Agra__AC Sleeper__Tier_A`

| Factor | Type | Why Used |
|---|---|---|
| Route | Hard filter | Buses on different routes are never comparable |
| Bus Type | Hard filter | AC Sleeper ≠ Non-AC Seater — different segments |
| Rating Tier | Soft group | Tier_A (≥4.2), Tier_B (3.5–4.1), Tier_C (<3.5) |

> **Why 3 factors and not 4?**  
> Adding Departure Time Bucket created 174 groups from 300 rows (avg 1.7 buses/group).  
> A median from 1–2 values is statistically unreliable.  
> With 3 factors: 60 groups, avg **8.3 buses each** — robust benchmarking.

---

### Step 2 — Benchmark Pricing

For each peer group, the **median price** is calculated as the benchmark.

> **Why median and not average?**  
> One premium bus charging ₹10,000 would skew the average up and make  
> every other bus appear underpriced. The median ignores outliers entirely.

```python
price_variance_pct = (our_price - group_median) / group_median × 100
price_gap_inr      = our_price - group_median
```

---

### Step 3 — Flag Pricing Issues

| Flag | Condition | Business Meaning |
|---|---|---|
| 🔴 Too High | Variance > +20% | Losing bookings to cheaper competitors |
| 🟡 Slightly High | Variance +10% to +20% | Monitor — approaching danger zone |
| 🟢 Fair | Variance within ±10% | Competitively priced |
| 🟡 Slightly Low | Variance -10% to -20% | Leaving some revenue on the table |
| 🔴 Too Low | Variance < -20% | Significant revenue leakage — raise urgently |

---

## 📈 Charts

### EDA Overview
![EDA Charts](outputs/eda_charts.png)

### Price Flagging Analysis
![Flagging Analysis](outputs/flagging_analysis.png)

### Deep Dive
![Deep Dive](outputs/deep_dive_charts.png)

---

## 🚀 How to Run

### Option A — Google Colab (Recommended, no setup needed)
1. Click the **Open in Colab** badge at the top of this README
2. Go to **Runtime → Run All**
3. To use your own data: replace the sample data block in **Cell 3** with `df = pd.read_csv('your_file.csv')`

### Option B — Run Locally

```bash
# Clone the repository
git clone https://github.com/[YOUR GITHUB USERNAME]/bus-price-benchmarking-system.git
cd bus-price-benchmarking-system

# Install dependencies
pip install pandas numpy matplotlib seaborn openpyxl

# Launch notebook
jupyter notebook notebook/Bus_Price_Benchmarking.ipynb
```

---

## 🔧 Configuration

All key settings are in **Cell 2** of the notebook:

```python
OUR_OPERATOR_NAME   = 'OurBus'   # ← Change to your operator name
THRESHOLD_HIGH      =  20        # Flag as Too High if >20% above median
THRESHOLD_MID_HIGH  =  10        # Flag as Slightly High if >10%
THRESHOLD_MID_LOW   = -10        # Flag as Slightly Low if <-10%
THRESHOLD_LOW       = -20        # Flag as Too Low if <-20%
MIN_PEER_GROUP_SIZE =  2         # Min buses in a group for valid comparison
```

---

## 🤖 MVP Automation Plan

| Stage | Tool | Action |
|---|---|---|
| 1 — Ingest | Google Sheets / S3 / API | Auto-pull daily booking data |
| 2 — Process | Python .py + GitHub Actions | Run pipeline on schedule (daily 7 AM) |
| 3 — Store | gspread → Google Sheets | Auto-write flagged output |
| 4 — Visualise | Looker Studio (free) | Live dashboard with route/flag filters |
| 5 — Alert | smtplib / Slack webhook | Email or Slack alert when flags spike |

---

## 📦 Deliverables

| File | Description |
|---|---|
| `excel/Bus_Price_Benchmarking_Assignment.xlsx` | Primary deliverable — 3 sheets: Flagging Output, Logic Explanation, Summary Dashboard |
| `notebook/Bus_Price_Benchmarking.ipynb` | Full Python pipeline — 11 cells, fully documented |
| `docs/Bus_Price_Benchmarking_Project_Report.docx` | Detailed methodology, design decisions, findings |
| `outputs/action_required_flags.csv` | 73 OurBus routes flagged, sorted by severity |

---

## 🛠️ Tech Stack

- **Python 3.10** — core language
- **pandas** — data manipulation, groupby, merge
- **numpy** — numerical operations
- **matplotlib + seaborn** — 9 charts across 3 figure panels
- **openpyxl** — Excel workbook creation with formatting
- **Google Colab** — development environment

---

## 📝 Key Design Decisions

| Decision | Rationale |
|---|---|
| Median over mean | Resistant to outlier prices — one extreme bus won't skew the benchmark |
| 3-factor key (not 4) | Keeps groups large enough (avg 8.3/group) for reliable median calculation |
| Load% excluded | It's an outcome of pricing, not a bus attribute — using it mixes cause and effect |
| ±10%/±20% thresholds | Business heuristics — should be calibrated against conversion data in production |
| Min peer group = 2 | Ensures 100% of OurBus routes are actionable (0 Insufficient Data rows) |

---

## 👤 Author

**[Harshit Choubey]**  
Data & Growth Intern Applicant  

