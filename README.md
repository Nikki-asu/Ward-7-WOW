# Ward-7-WOW
Ward 7 WOW Factor Analysis — Toronto 311 service request signal detection for Ward 7. Produces rising/drifting signals, hotspot maps, city comparisons, and a narrative summary. Built on an event → measure → signal ETL architecture for KinesisIQ 


---

##  Overview

This project analyzes Toronto Open Data 311 service requests for Ward 7 to produce actionable signals for city councillors and civic planners. Rather than just counting complaints, it detects what is **rising**, what is **drifting**, what is **unusual**, and what is **likely to escalate** — all tied back to a reusable ETL pipeline.

All category bucketing and signal rules are consistent with the team config taxonomy, so every insight flows from **event → measure → signal** and is not a one-off analysis.

---

##  File Structure

```
ward-7-analysis/
│
├── generate_ward7_wow.py      # Main analysis script — run this
├── capture_report.py          # Category rule definitions (EXACT_RULES)
├── .gitignore                 # Ignores outputs, data, venv, pycache
│
└── ward7_wow_outputs/         # Generated on run — gitignored
    ├── plots/                 # All .png chart files
    ├── data/                  # All .csv data files
    └── ward7_story.txt        # Narrative signal summary
```

> Raw SR data CSVs and all generated outputs are gitignored. You will need to supply your own data files and update `DATA_DIR` in the script.

---

## ⚙️ Configuration

All key parameters are defined at the top of `generate_ward7_wow.py`:

| Variable | Default | Description |
|---|---|---|
| `DATA_DIR` | `C:\Users\nikki\...\data` | Path to folder containing SR CSVs |
| `FULL_YEARS` | 2018–2025 | Years included in main analysis |
| `BASELINE_YEARS` | 2018–2024 | Historical baseline window |
| `RECENT_YEARS` | 2025 | Recent period for annual signal detection |
| `RECENT_MONTHS` | 3 | Recent window for monthly analysis |
| `BASELINE_MONTHS` | 12 | Baseline window for monthly analysis |
| `DRIFT_MONTHS` | 6 | Window for location drift slope calculation |
| `MIN_REPEAT_THRESHOLD` | 3 | Min occurrences to flag a repeated complaint |
| `TOP_N_RISING` | 5 | How many rising categories to surface |
| `TOP_N_DRIFTING` | 3 | How many drifting locations to surface |
| `TOP_N_HOTSPOTS` | 5 | How many hotspot micro-areas to surface |

To run for a different ward, change `TARGET_WARD_NUM = "07"` — the regex filter is parameterized for this purpose.

---

## 🚀 How to Run

```bash
# 1. Activate your environment
python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # Mac/Linux

# 2. Install dependencies
pip install pandas numpy matplotlib scipy

# 3. Update DATA_DIR in generate_ward7_wow.py to point at your SR CSVs

# 4. Run
python generate_ward7_wow.py
```

---

## 📊 Outputs

All outputs are written to `ward7_wow_outputs/` which is gitignored.

### Charts (`plots/`)

| File | Description | Time Window |
|---|---|---|
| `yoy_trends.png` | Category trends year over year | 2018–2025 |
| `ward7_category_trends_monthly.png` | Monthly category trends | 2018–2025 |
| `ward7_category_heatmap.png` | Heatmap of category volume by month | 2018–2025 |
| `top_5_rising_annual.png` | Top 5 rising categories (annual) | 2025 vs 2018–2024 |
| `top_5_rising_monthly.png` | Top 5 rising categories (monthly) | Last 3 months vs prior 12 |
| `top_3_drifting_locations_trend.png` | Top 3 drifting micro-locations | Last 6 months slope |
| `categories_faster_than_city.png` | Ward 7 growth vs city growth | Last 3 months vs prior 12 |
| `repeated_complaint_locations.png` | Repeated complaints by location | Last 12 months |
| `repeated_complaints_stacked.png` | Stacked repeated complaints by type | Last 12 months |
| `ward7_vs_city_trend.png` | Ward 7 vs citywide category share | 2018–2025 |
| `ward7_vs_city_ratio.png` | Ward 7 / city proportion ratio | 2018–2025 |
| `ward7_vs_its_own_history.png` | Ward 7 vs its own historical baseline | Last 3 months vs prior 12 |
| `top_5_hotspots.png` | Top 5 complaint hotspot micro-areas | Last 6 months |

### Data (`data/`)

| File | Description |
|---|---|
| `top_5_rising_annual.csv` | Annual rising category stats |
| `top_5_rising_monthly.csv` | Monthly rising category stats |
| `top_3_drifting_locations.csv` | Drifting location slopes |
| `categories_faster_than_city.csv` | Ward 7 vs city growth rates |
| `repeated_complaint_locations.csv` | Repeated complaint counts by location + type |
| `ward7_vs_city_baseline.csv` | Ward 7 / city proportion ratios |
| `ward7_vs_its_own_history.csv` | Historical baseline comparison |
| `third_consecutive_month_risers.csv` | Categories rising 3 consecutive months |
| `top_5_hotspots.csv` | Hotspot micro-area data |
| `early_warning_2026.csv` | 2026 partial year early warning flags |

### Narrative

| File | Description |
|---|---|
| `ward7_story.txt` | Plain-text signal summary — rising, drifting, hotspots, early warnings |

---

## Signal Detection

The analysis uses a **dual-gate** approach to avoid false positives:

- **RISING** — percent change > 10% **AND** z-score > 1.0 (change is unusual given that category's normal volatility)
- **DRIFTING** — slope is positive and percent change > 5% (softer signal, worth watching)
- **FALLING** — percent change < -10% **AND** z-score < -1.0
- **STABLE** — everything else

Z-score is key here. A category like Roads naturally spikes in winter, so a large percent change isn't alarming on its own. Z-score anchors the signal to how much that category **normally** varies.

---

## ETL Layers

| Layer | What it does |
|---|---|
| **Event** | Raw 311 service request rows — timestamped, located, typed |
| **Measure** | Category bucketing via `EXACT_RULES` — turns raw SR types into structured counts |
| **Signal** | Change detection — z-score, slope, percent change, hotspot ranking |

---

## Dependencies

```
pandas
numpy
matplotlib
scipy
```
