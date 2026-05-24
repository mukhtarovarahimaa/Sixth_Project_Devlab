# Sixth_Project_Devlab

# 📝 E-Commerce Funnel Analysis — Methodology & Growth Insights

**Project:** User Behavior Funnel Analysis
**Dataset:** [eCommerce Events History in Cosmetics Shop](https://www.kaggle.com/datasets/mkechinov/ecommerce-events-history-in-cosmetics-shop)  
**Period:** January 2020 (~2.6M rows) + February 2020 (~1.7M rows)  


---

## 🔬 Dataset Description

| Column | Description |
|---|---|
| `event_time` | UTC timestamp of the event |
| `event_type` | One of: `view`, `cart`, `purchase` |
| `product_id` | Unique product identifier |
| `category_id` | Numeric category identifier |
| `category_code` | Dot-separated category hierarchy (e.g. `electronics.smartphone`) |
| `brand` | Brand name (nullable) |
| `price` | Product price in USD |
| `user_id` | Unique user identifier |
| `user_session` | Session identifier |

---

## 🛠 Methodology

### 1. Data Preparation
- Both CSV files loaded and concatenated with a `month` label column for traceability.
- Missing `user_id` rows dropped (cannot track funnel without identity).
- Exact duplicate events (same user + product + event + timestamp) removed.
- Missing `category_code` values filled with `"unknown"`.
- Top-level category extracted: `electronics.smartphone` → `electronics`.
- Price column coerced to numeric; non-numeric values become NaN.

### 2. Funnel Logic
The funnel counts **unique users** at each stage — not event occurrences:
- **Stage 1 (View):** Users who triggered at least one `view` event.
- **Stage 2 (Cart):** Users who triggered at least one `cart` event.
- **Stage 3 (Purchase):** Users who triggered at least one `purchase` event.

> **Important:** These are independent counts. A user who purchased without an explicit cart event (e.g. direct checkout) would appear in Stage 3 but not Stage 2. This is intentional — it reflects actual funnel penetration.

### 3. Conversion Rate Formulas
```
View → Cart Rate     = Cart Users / Viewer Users × 100
Cart → Purchase Rate = Purchaser Users / Cart Users × 100
Overall Conv Rate    = Purchaser Users / Viewer Users × 100
```

### 4. Category Breakdown
Top 8 categories selected by view-user count. Each category's funnel computed independently (same 3-stage logic). This ensures category comparisons are not distorted by shared users across categories.

### 5. Monthly Comparison
January and February data separated before funnel computation. Conversion rate delta calculated as `February rate − January rate` in percentage points (pp).

### 6. Brand Analysis
Filtered to rows with non-null `brand`. Top 10 brands ranked by unique purchasers. View-to-purchase rate computed per brand using the same unique-user logic.

### 7. Price Outlier Detection (IQR)
Applied to purchase events only:
```
IQR   = Q3 − Q1
Fence = [Q1 − 1.5×IQR, Q3 + 1.5×IQR]
Outliers = prices outside [Lower Fence, Upper Fence]
```

### 8. Time-to-Purchase (Bonus)
For users who both viewed and purchased:
- First view timestamp = `min(event_time)` where `event_type == 'view'`
- First purchase timestamp = `min(event_time)` where `event_type == 'purchase'`
- Elapsed time = purchase − view (only positive gaps kept)

### 9. High-View / Low-Conversion Products (Bonus)
- "High-view" = products in top 20% by unique viewer count.
- "Low-conversion" = view-to-purchase rate below 2%.
- These products attract traffic but fail to convert — prime optimisation targets.

---

## 📊 Key Metrics Reference

| Metric | Value |
|---|---|
| Total Events (combined) | ~4.2M |
| Event split: view | ~85% |
| Event split: cart | ~9% |
| Event split: purchase | ~6% |
| **View → Cart Rate** | **~10.2%** |
| **Cart → Purchase Rate** | **~71.4%** |
| **Overall Conversion Rate** | **~7.3%** |
| Feb vs Jan conversion improvement | **+0.8 pp** |
| Highest-converting category | **accessories (~12.1%)** |
| Top 5 brands' purchase share | **~34%** |
| Median cart value | **~$26.8** |

---

## 🌱 Growth Insights for Product & Growth Teams

### Insight 1 — View-to-Cart Is the Largest Leakage Point
Only ~10% of viewers add a product to cart. This is the funnel's biggest gap. Product pages need stronger conversion signals: urgency indicators (low stock), clearer CTAs, and social proof (reviews, ratings count).

### Insight 2 — Protect the Cart Stage (Intent Is Already There)
~71% of cart users complete the purchase. This is high, but losing ~29% of already-committed users at checkout is expensive. Audit checkout UX: are there surprise fees, slow load times, or too many steps?

### Insight 3 — February Quality Improvement
Despite lower traffic volume in February, conversion rate rose ~0.8 pp. This suggests either better traffic quality (improved targeting) or a product/pricing change that resonated. Identify and replicate the driver.

### Insight 4 — High-View / Low-Conv Products = Catalogue Debt
A segment of well-discovered products fails to convert. The problem is not reach — it's persuasion. These products need richer content (better images, detailed descriptions, size guides) and possibly price testing.

### Insight 5 — Price Outliers Pollute AOV Metrics
Outlier purchase prices (some reaching $1000+ in a cosmetics store) are almost certainly data quality issues. If these flow unchecked into dashboards, they inflate Average Order Value and mislead pricing strategy. Implement automated price validation rules upstream.

---

## 📁 Output Files

| File | Description |
|---|---|
| `ecommerce_funnel_analysis.ipynb` | Full analysis notebook |
| `note.md` | This methodology document |
| `01_event_distribution.png` | Event type distribution (bar + donut) |
| `02_funnel_chart.png` | User funnel (view → cart → purchase) |
| `03_category_conversion.png` | Conversion rates by top 8 categories |
| `04_monthly_comparison.png` | January vs February funnel comparison |
| `05_top_brands.png` | Top 10 brands by purchases + conversion rate |
| `06_price_outliers.png` | Price distribution + IQR outlier fences |
| `07_time_to_purchase.png` | Time elapsed from first view to purchase |
| `08_high_view_low_conv.png` | High-traffic, low-converting product scatter |
