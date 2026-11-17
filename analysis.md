# Fashion Sales — Dataset Guide and Analysis

*Source file: `fashion_sales.csv` · 240 data rows + 1 header · 14 columns*
*Charts referenced below live in `charts/`.*

---

## Part 1 — Dataset guide (for a first-time reader)

### What it covers

`fashion_sales.csv` is **two years of monthly sales performance for a mid-market fashion retailer**: January 2024 through December 2025 (24 months). The retailer sells **five product categories** through **two channels**.

- **Categories:** Outerwear, Dresses, Footwear, Denim, Accessories
- **Channels:** Online, Store
- **Prices:** list prices run $35–$180; discounts run 0%–40%
- **Totals across the whole file:** ~$13.22M revenue, 202,500 units, $2.34M ad spend

### How it is organized

The file is a **complete, regular grid** — a summary table, not a transaction log.

- **One row = one month × one category × one channel.** For example, row 2 is "Dresses, sold Online, in January 2024."
- The grid is fully populated: **24 months × 5 categories × 2 channels = 240 rows**, with **exactly 10 rows per month** and **no duplicate combinations**.
- Rows are sorted chronologically, then by category, then channel.

Because rows are already **aggregated**, each figure (units, revenue, ad spend) is a monthly total for that category/channel slice. There is no customer-level, order-level, or product-level detail anywhere in the file.

### Column dictionary

| Column | Type | Meaning | Range / values |
|---|---|---|---|
| `month` | text | Year-month label | 2024-01 … 2025-12 |
| `year` | int | Calendar year | 2024, 2025 |
| `month_num` | int | Month number 1–12 | 1–12 |
| `category` | text | Product category | Accessories, Denim, Dresses, Footwear, Outerwear |
| `channel` | text | Sales channel | Online, Store |
| `avg_temp_F` | float | Avg temperature that month (°F) | 31.3–80.4 |
| `list_price` | int | Regular (pre-discount) price ($) | 35, 70, 85, 110, 180 |
| `discount_pct` | int | Discount applied (%) | 0, 10, 15, 20, 25, 30, 40 |
| `sale_price` | float | Realized selling price ($) | 19.57–197.18 |
| `ad_spend` | int | Advertising spend ($) | 2,050–19,270 |
| `avg_rating` | float | Average product rating | 3.4–5.0 |
| `units_sold` | int | Units sold | 220–2,488 |
| `revenue` | float | Revenue ($) | 17,257–154,512 |
| `return_rate` | float | Fraction of units returned | 0.01–0.356 (1%–35.6%) |

### Which columns depend on each other

Some columns are **derived from others** — they are not independent measurements:

1. **`revenue` = `sale_price` × `units_sold` — exactly, in all 240 rows.** This is an identity, not a coincidence. If you know the price and the units, you know the revenue.

2. **`sale_price` ≈ `list_price` × (1 − `discount_pct`/100), but with noise.** In **0 of 240 rows** does the sale price equal the clean "list minus discount" value. Instead the actual value is the expected price multiplied by a random factor of roughly **0.90 to 1.10** (mean 0.998, sd 0.059) — i.e. the sale price carries a ±10% random jitter.
   *Example:* Dresses, list $85, 30% off → "clean" $59.50, but actual `sale_price` is $54.31 (−8.7%).

3. **`list_price` is fixed per category** (one price per category across all rows): Accessories $35, Denim $70, Dresses $85, Footwear $110, Outerwear $180.

4. **`avg_temp_F` is constant within each month** (one temperature per month, shared by all 10 rows of that month). It describes the month, not the category or channel.

5. **`year`, `month_num` are derived from `month`** and always agree with it.

6. **`avg_rating`, `units_sold`, `ad_spend`, `return_rate`, `discount_pct`** are independent per-row values — not derived from other columns.

### Data quality and surprises

- **Clean file:** no missing values, no duplicate rows, no non-numeric entries, all dates/years consistent, `return_rate` and `discount_pct` always within sensible bounds.
- **Surprising #1 — the price relationship is noisy, not exact.** Anyone expecting `sale_price` to be a deterministic function of list price and discount will be surprised; the ±10% jitter means `sale_price` behaves like its own variable.
- **Surprising #2 — the file looks synthetically generated.** The uniform ±10% price noise and an implausibly strong channel→returns link (see below) suggest this is simulated data, so treat the patterns as clean structure rather than real market behavior.
- **Surprising #3 — "biggest seller" and "biggest earner" are different categories.** Accessories sells the most units but earns the least revenue; Outerwear is the reverse.
- **Surprising #4 — returns are overwhelmingly a channel story.** Online returns run ~2.5x Store returns, and the split holds inside every category.

---

## Part 2 — Full analysis

All figures below are computed from the 240 rows. "Units-weighted" means each row's rate is weighted by its `units_sold`, which reflects the true share of units.

### 2.1 Seasonality

**Outerwear and Dresses are strongly seasonal; the rest are only mildly so.**

| Category | corr(units, temp) | slope (units/°F) | Peak month | Trough month |
|---|---:|---:|---|---|
| Outerwear | **−0.77** | −16.3 | Dec (1,243) | Jun (312) |
| Dresses | **+0.62** | +18.6 | Jul (2,160) | Feb (479) |
| Footwear | −0.22 | −3.2 | Dec | Apr |
| Denim | −0.27 | −4.4 | Dec | Apr |
| Accessories | −0.25 | −3.9 | Dec | Feb |

- Outerwear sells **~4.0x more in December than June**.
- Dresses show the mirror image: **July is ~4.5x February**.
- Footwear, Denim and Accessories all peak in **December** but correlate weakly with temperature — their seasonality is holiday-driven, not weather-driven.
- Aggregate monthly revenue vs temperature: **r = −0.39** (winter edges summer, but only mildly, because the two biggest categories pull in opposite directions).

### 2.2 2024 vs 2025

| Metric | 2024 | 2025 | Change |
|---|---:|---:|---:|
| Revenue | $6,406,734 | $6,813,748 | **+6.4%** |
| Units sold | 97,712 | 104,788 | **+7.2%** |
| Revenue per unit | $65.57 | $65.02 | −0.8% |
| Avg discount | 22.9% | 22.3% | −0.6 pt |
| Ad spend | $1,098,180 | $1,239,350 | **+12.9%** |
| Ad spend as % of revenue | 17.1% | 18.2% | +1.1 pt |
| Return rate (units-wtd) | 17.2% | 17.9% | **+0.7 pt** |
| Avg rating | 4.238 | 4.200 | −0.038 |

- Growth was **broad-based** — every category grew, led by Dresses (+8.7%), then Accessories (+6.7%), Denim (+6.1%), Outerwear (+5.5%), Footwear (+5.1%).
- But **ad spend grew twice as fast as revenue** (+12.9% vs +6.4%), while returns rose and ratings dipped slightly. 2025 bought growth with more marketing, not better economics.

### 2.3 Temperature effects by category

- **Outerwear — cold-weather product** (r = −0.77): each 1°F warmer costs ~16 units.
- **Dresses — warm-weather product** (r = +0.62): each 1°F warmer adds ~19 units.
- **Footwear, Denim, Accessories — largely temperature-insensitive** (|r| ≤ 0.27); steadier, holiday-influenced demand.

### 2.4 Do bigger discounts lead to more units sold?

**Yes — one of the strongest, cleanest patterns in the data.**

- Overall **corr(discount, units) = +0.51**; regression slope **+13.3 units per additional discount point**.
- Holds **within every category** (so it isn't a category artifact): Footwear r = +0.69, Denim +0.75, Outerwear +0.57, Dresses +0.55, Accessories +0.55.
- Mean units rise almost monotonically with discount:

| Discount | 0% | 10% | 20% | 30% | 40% |
|---|---:|---:|---:|---:|---:|
| Mean units | 586 | 607 | 784 | 911 | **1,129** |

- Discounts also raise **revenue**, not just units: mean revenue goes from **$48.3K at 0% off to $63.0K at 40% off** (corr(discount, revenue) = +0.20). Volume gain outweighs the price cut here.
- *Caveat:* observational — high-demand periods may be *given* bigger discounts, so some of this is reverse causality rather than pure price elasticity.

### 2.5 Does ad spending pay off?

**Yes, with sharply diminishing returns by category — and the headline correlation understates it.**

- Naively weak: corr(ad, revenue) = **0.21**, corr(ad, units) = 0.25.
- That's a **confounding artifact** (Accessories gets average spend but low revenue). Within categories the link is real: pooled within-category corr(ad, revenue) = **0.42** (Footwear 0.61, Denim 0.56).

| Category | ROI (rev/ad) | Ad spend as % of revenue |
|---|---:|---:|
| Outerwear | **8.52** | 11.7% |
| Dresses | 6.57 | 15.2% |
| Footwear | 6.18 | 16.2% |
| Denim | 4.25 | 23.5% |
| Accessories | **2.81** | **35.6%** |

- **Accessories spends 35.6% of revenue on ads for only $2.81 back per dollar** — the least efficient category by far.
- By channel: **Store ROI 6.27 > Online 5.20**; Online spends more of its revenue on ads (19.2% vs 15.9%).

### 2.6 What is related to returns?

**Channel is by far the dominant driver.**

- corr(return_rate, "Online" dummy) = **+0.85** overall, and **0.95–0.97 within every category**. Online mean return rate **24.7% vs Store 10.0%** — a 2.5x gap.

| Category | Online | Store |
|---|---:|---:|
| Outerwear | 21.1% | 7.0% |
| Dresses | 28.9% | 14.9% |
| Footwear | 29.7% | 14.7% |
| Denim | 22.3% | 6.6% |
| Accessories | 21.2% | 6.5% |

- The gap holds in **all five categories** — it's the channel, not one bad category (consistent with buying online without trying items on).
- Secondary driver is **category type**: fit-sensitive Dresses and Footwear (~22.4%) return more than the rest (~14%).
- Returns are **not** meaningfully tied to discount (r = −0.10) or temperature (r ≈ 0.00). Ad spend shows a weak positive link (r = +0.23) that is almost certainly the channel/category confound.

### 2.7 What is related to ratings?

**Uniformly high; strongest (still modest) correlate is returns.**

- Range only 3.4–5.0; category means span a tiny band (Accessories 4.25 down to Denim 4.18). Channel means: Store **4.29** vs Online **4.15**.
- **corr(rating, return_rate) = −0.35**, holding within every channel (Online −0.27, Store −0.29) and every category (−0.26 to −0.45): more returns → lower ratings.
- Weak positive link to volume: corr(rating, units) = +0.13.
- Essentially **no relationship** with price (r = −0.01), discount (r = −0.01) or temperature (r = 0.00).

---

## Part 3 — Caveats

- **Correlation ≠ causation.** These are 240 aggregated monthly rows, not transactions, so no experiment isolates a single cause.
- **The data looks synthetic.** `sale_price` = list × (1 − discount) × random(0.90–1.10) in 100% of rows, and the channel→return link is implausibly strong (r ≈ 0.95 within category). Read the patterns as clean structure, not real market behavior.
- **No granularity** below month × category × channel is available.

## Bottom line

Outerwear is a profitable winter engine (ROI 8.5), Dresses a summer engine, discounts reliably lift volume and even revenue, and the Store channel is both more ad-efficient and dramatically better on returns. The two clearest problems to attack are the **Online return rate (~25%, 2.5x Store)** and **Accessories' poor ad ROI ($2.81 per dollar)**.
