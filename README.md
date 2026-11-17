# TQQQ Historical All-Time Highs & Drawdown Dashboard

A standalone, interactive HTML dashboard for analyzing **daily closing-price all-time highs (ATHs)** and **drawdowns** for ProShares UltraPro QQQ (`TQQQ`). Built with vanilla HTML, CSS, and JavaScript; no framework, installation, database, or API key is required.

The dashboard can serve as a **reusable template for other tickers** when supplied with comparable daily historical price data.

## Project files

```text
tqqq-analysis/
├── index.html              # Dashboard (rename tqqq_all_time_highs_dashboard_v4.html)
├── data/
│   └── tqqq_us_d.csv       # Original price data for reproducibility
└── README.md
```

> **Important:** The current `index.html` contains a **snapshot of the TQQQ closing-price history embedded in its JavaScript**. The separate CSV is retained as an auditable source file. **Updating the CSV in GitHub alone does not update the embedded dashboard.** You can load a different CSV through the dashboard's upload control for that browser session, or regenerate/replace its embedded data for a permanent update.

## Dashboard features

1. **Price and ATH overview** — total new closing-price records, earliest/latest ATH dates, longest completed gap, historical price series, and record markers.
2. **Sortable ATH history** — date, record close, previous record close, percentage change, calendar days between records, and observed trading sessions between records; year filter and CSV export.
3. **Annual ATH frequency** — vertical year-versus-count chart and horizontal annual count bars. **Zero-ATH years are included** (including 2022 and 2023 in this dataset).
4. **Historical and current drawdowns** — underwater chart, drawdown as of the CSV's final date, worst recorded drawdown, and major drawdown episodes showing peak/trough/recovery information. Unrecovered episodes are marked ongoing.

## Input CSV format

The loader requires **`Date`** and **`Close`** column headers (case-insensitive). Other columns are allowed and ignored by the analysis.

```csv
Date,Open,High,Low,Close,Volume
2010-02-11,0.193005,0.206296,0.192411,0.205155,7281103.61075496
2010-02-12,0.199638,0.207795,0.198451,0.206058,18123858.5438875
```

Requirements:

- One observation per trading date; ISO dates (`YYYY-MM-DD`) are recommended.
- Positive numeric closing prices; no duplicate dates.
- Adjusted/split-consistent prices are strongly recommended for long-term comparisons. **Use the same adjustment convention throughout the file.** The dashboard uses the provided `Close` values exactly; it does not perform split or dividend adjustments itself.
- Rows can be uploaded in any order; the loader sorts them by date and skips invalid date/price rows (reporting the count). Duplicate dates produce an error.
- A ticker's full available history is preferred. ATH and drawdown values are **relative to the start of the uploaded sample**; a truncated history can produce misleading records.

## Definitions and calculations

### All-time highs

A **new closing-price ATH** occurs only when a day's close **strictly exceeds** every earlier closing price in the supplied sample. Equal closes do not count as new records.

```text
running_peak[t] = max(Close[0], ..., Close[t])
new_ATH[t] = Close[t] > max(Close[0], ..., Close[t-1])
ATH_increase_% = (new_ATH_close / previous_ATH_close - 1) * 100
calendar_gap = new_ATH_date - previous_ATH_date (calendar days)
trading_gap = index(new_ATH) - index(previous_ATH) (observed sessions)
```

The first valid date establishes the initial record and has no prior record or gap. Annual ATH totals group record dates by calendar year, **including zeros** for years covered by the dataset.

### Drawdowns

A drawdown measures a day's closing price relative to the highest close observed **up to that date**:

```text
drawdown_%[t] = (Close[t] / running_peak[t] - 1) * 100
current_drawdown = drawdown_% on the CSV's latest date
maximum_drawdown = min(drawdown_% across all included dates)
```

A drawdown episode begins when the price drops below its prior closing peak. Its trough is the lowest relative point within that episode. A recovery occurs when a later **close equals or exceeds** the prior peak. An episode without a recovery by the final CSV date remains **ongoing**. The underwater chart displays zero at/above prior records and negative percentages below them.

**Terminology:** “Current” means **current as of the latest date in the CSV**, not a live market quote. All metrics are close-to-close, not intraday.

## Run locally

1. Download or clone the repository.
2. Open `index.html` directly in Chrome, Edge, or another modern browser.
3. Use the bottom navigation arrows (or keyboard left/right arrows) to move through the four sections.
4. To analyze an alternative CSV **temporarily**, select the dashboard's upload control and choose the file. Charts, ATH history, and drawdowns will recalculate in the browser.
5. Export the ATH report from the dashboard when needed.

No local server is necessary for the **embedded-data version**.

## Recreate the dashboard for another ticker

The simplest approach is to **copy the existing HTML as a template** and keep the same CSV format and calculations:

1. **Obtain historical daily prices** for the new ticker, ideally from inception through the desired final date, with reliable split-adjusted closing prices. Export as CSV containing `Date` and `Close`.
2. **Inspect and validate the data**: check header names, nonpositive/missing closes, duplicate dates, splits, adjustment methodology, and date coverage.
3. **Make a ticker-specific copy** of `index.html`, for example `qqq/index.html`, and put the source CSV in `qqq/data/qqq_us_d.csv`.
4. **Test the new dataset immediately** using the dashboard's CSV upload control. Confirm the resulting counts, chart axes, year range, and latest date. This preview is not saved into the HTML.
5. **For a permanent, self-contained ticker dashboard**, replace the original embedded closing-price array in the HTML with the new ticker's `Date`/`Close` values, keeping the shape `[["YYYY-MM-DD", 123.45], ...]`. The embedded array appears near the end of the HTML in the script labeled `Embedded close-price history`. Keep the existing initialization/analysis logic intact. An AI coding assistant or a small build script can automate this substitution.
6. **Update ticker-specific text**: page title, visible headings, labels, comments, filename, and any references to `TQQQ`. Rename the ticker-specific source CSV and update this README accordingly.
7. **Reload and verify** the self-contained HTML **without uploading any CSV**. Its default results should match the values you saw in step 4.
8. **Commit the generated HTML, CSV, and README** to GitHub. If publishing through GitHub Pages, use `index.html` as the entry point.

### Suggested reusable AI coding prompt

```text
Use this repository's index.html as the source template and the new ticker CSV
as the authoritative input. Create a standalone index.html for ticker [TICKER]
with the same four-section layout, styling, ATH reports, sortable headers,
annual chart (including zero-ATH years), underwater drawdown chart, and
recovery table. Preserve the existing formulas and date conventions.

Validate the CSV Date/Close columns, sort dates, reject duplicate dates,
and embed the new daily Date/Close data into the HTML so the file works
offline with no upload. Replace all ticker-specific labels and metadata.

Verify that the embedded dashboard initializes on load; ATH counts equal
the sum of annual counts; current drawdown equals latest_close /
running_peak - 1; worst drawdown equals the minimum underwater value;
and that the year filter, table sorting, CSV export, and navigation work.
Do not silently substitute market data or invent missing prices.
```

## Updating an existing ticker

When newer pricing data becomes available, replace the source CSV and **rebuild/re-embed** the daily closing-price array in the HTML. Verify that the date range and current drawdown change as expected, and then commit both updated files together.

For a future multi-ticker version, consider moving the data out of the HTML and loading versioned CSV/JSON data files via `fetch()` from GitHub Pages. This would require refactoring the current embedded-data architecture and running the dashboard through a web server; it is **not** how this version currently works.

## Quality checks before publishing

- [ ] Date range and total valid price rows match the CSV.
- [ ] ATH dates/prices are strictly increasing in record price; equal closes are not counted twice.
- [ ] Sum of annual ATH counts equals total ATH count; empty years display `0`.
- [ ] ATH table sorts correctly in ascending and descending order; year filter and export work.
- [ ] Underwater values are never positive and return to `0` at a recovered record.
- [ ] Drawdown snapshot corresponds to the CSV's final date (not today's live price).
- [ ] Worst drawdown and peak/trough/recovery dates match independent calculations.
- [ ] Ticker name, source-data description, adjustment convention, and last dataset date are documented.

## Notes and limitations

- This project is for exploratory historical analysis, **not investment advice or a return forecast**.
- Leveraged ETFs such as TQQQ have path-dependent performance and can experience extreme drawdowns.
- Comparisons across tickers depend on the quality and consistency of their adjusted close histories.
- The dataset may contain revised historical prices or different adjustment rules depending on the data vendor.
- Hosting source prices in a public repository means the CSV is public; check the provider's license/redistribution terms before publishing a downloaded dataset.

## Technology

**HTML5 / CSS3 / vanilla JavaScript / inline SVG** — no framework, backend, external charting library, or API key.
