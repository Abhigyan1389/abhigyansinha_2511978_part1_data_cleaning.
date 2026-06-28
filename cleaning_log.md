# 📋 Data Quality Report — raw_orders.xlsx

**Total Records:** 932 | **Columns:** 21 | **Quality Dimensions Assessed:** 7 | **Report Date:** June 2026

---

## Overall Data Health Scorecard

| Metric | Value |
|---|---|
| Total Records | 932 |
| ✅ CLEAN Records | 548 |
| ⚠ WARNING Records | 347 |
| ✗ INVALID Records | 37 |
| Exact Duplicates Removed | 20 |
| Fields with Missing Values | 3 |
| Ship Date Anomalies | 21 |
| Sales / Profit Mismatches | 46 |

**Record Quality Distribution**

| Flag | Count | Share |
|---|---|---|
| ✅ CLEAN | 548 | 58.8% |
| ⚠ WARNING | 347 | 37.2% |
| ✗ INVALID | 37 | 4.0% |

---

## 1. Missing Value Summary

> Only 3 of 21 columns contain missing values. All handled via business rules — no records deleted.

| Field | Missing Count | % of Total | Business Rule | Action Taken | Status |
|---|---|---|---|---|---|
| `region` | 26 | 2.8% | Fill blank → "Unknown" | Filled; added to quality flag | ⚠ WARNING |
| `ship_mode` | 22 | 2.4% | Fill blank → "Unknown" | Filled; added to quality flag | ⚠ WARNING |
| `discount` | 26 | 2.8% | Fill 0 if sales fields valid | All 26 filled as 0 (valid sales) | ⚠ WARNING |

**Summary:** 3 of 21 fields affected — all missing values resolved, 0 records deleted.

---

## 2. Duplicate Summary

> Exact copies removed. Conflicting order_ids retained and flagged — no silent deletion.

| Type | Unique Order IDs | Rows Affected | Action | Impact on Dataset | Status |
|---|---|---|---|---|---|
| Exact duplicate rows | 19 | 20 | Remove — keep first occurrence | Zero info lost; clean dataset reduced to 912 | ✗ Removed |
| Conflicting order_ids | 12 | 25 | Retain both; flag Primary / Secondary | Requires analyst review — do not auto-delete | ⚑ Flagged |

**Summary:** 20 rows removed | 25 rows flagged for analyst review | Total touched: 45

---

## 3. Invalid Discount Summary

> Discount range: 0 (0%) to 1 (100%). Negatives are data entry errors. Missing filled as 0 where sales are valid.

| Category | Records | % of Total | Range Check | Action Taken | Status |
|---|---|---|---|---|---|
| Valid (0% – 100%) | 890 | 95.5% | 0 ≤ d ≤ 1 | Used as-is in calculations | ✅ CLEAN |
| Missing → Filled as 0 | 26 | 2.8% | Was blank | Filled 0; all 26 had valid sales fields | ⚠ WARNING |
| Negative (< 0%) | 16 | 1.7% | d < 0 | Flagged INVALID; excluded from `calc_sales` | ✗ INVALID |
| Above 100% (> 1.0) | 0 | 0.0% | d > 1 | None found — rule documented for future | ✓ None found |

**Summary:** Usable discounts (valid + filled): 916 | Invalid (negative): 16 | Above 100%: 0

---

## 4. Date Issue Summary

> 6 distinct date formats detected. All parsed and standardised to DD/MM/YYYY. 21 records have `ship_date` before `order_date`.

| Issue / Format | `order_date` | `ship_date` | Count | Resolution Applied | Status |
|---|---|---|---|---|---|
| ISO (YYYY-MM-DD) | Unambiguous | Unambiguous | — | Direct parse | ✓ OK |
| DD Mon YYYY | Unambiguous | Unambiguous | — | Direct parse | ✓ OK |
| Slash / dash (one part > 12) | Unambiguous | Unambiguous | — | Part >12 determines order | ✓ OK |
| Ambiguous (both parts ≤ 12) | 201 | 176 | 377 | Context-resolved using 0–60 day ship window | ⚠ WARNING |
| Ship date before order date | — | — | 21 | Flagged INVALID; excluded from `shipping_delay_days` | ✗ INVALID |

**Summary:** All dates standardised to DD/MM/YYYY | 21 invalid shipping records excluded from delay analysis.

---

## 5. Order Status Issue Summary

> Cancelled orders excluded from completed sales. Returned and Refunded orders have dedicated separate summaries.

| Order / Payment Status | Records | % of Total | Gross Sales (₹) | Treatment in Analysis | Status |
|---|---|---|---|---|---|
| Completed | 622 | 66.7% | 6,053,703.16 | Included in completed sales summary | ✅ CLEAN |
| Cancelled (order_status) | 146 | 15.7% | 1,458,700.65 | Excluded from completed sales; shown in separate table | ⚠ WARNING |
| Returned (order_status) | 164 | 17.6% | 1,632,404.84 | Separate returns summary — not in completed total | ⚠ WARNING |
| Refunded (payment_status) | 72 | 7.7% | 677,707.88 | Separate refunds summary (38 Cancelled + 34 Returned) | ⚠ WARNING |

**Summary:** Completed sales: ₹6,053,703.16 (622 records) | Cancelled ₹1,458,700.65 excluded | Refunded ₹677,707.88 isolated.

---

## 6. Sales / Profit Calculation Mismatch Summary

> Formula: `calculated_sales = Qty × Unit Price × (1 − clean_discount)`. 46 records differ from recorded sales by >₹1.

| Metric | Count | % of Total | Detail | Status |
|---|---|---|---|---|
| Records with INVALID discount | 16 | 1.7% | Negative discount → `calculated_sales` cannot be computed | ✗ INVALID |
| Sales mismatch (diff > ₹1) | 46 | 4.9% | Avg gap ₹1,992 — Max gap ₹16,803 | ⚠ WARNING |
| Profit mismatch (diff > ₹1) | 47 | 5.0% | Follows from sales mismatch — same root cause | ⚠ WARNING |
| Records matching exactly (diff ≤ ₹1) | 870 | 93.4% | Recorded = Calculated within rounding tolerance | ✅ CLEAN |
| Root cause | — | — | Wrong discount at data entry | ℹ INFO |

**Recommendation:** Use `calculated_sales` and `calculated_profit` columns for all financial analysis and reporting.

---

## 7. Final Record Count — CLEAN / WARNING / INVALID

> After applying all 7 business rules: missing fills, duplicate removal, discount validation, date checks, and order status rules.

| Flag Level | Count | % of Total | Triggered When | Analyst Action Required | Priority |
|---|---|---|---|---|---|
| ✅ CLEAN | 548 | 58.8% | All source fields valid; dates chronological; normal order and payment status | None — record is safe for all analysis and reporting | 🟢 Low |
| ⚠ WARNING | 347 | 37.2% | Missing values filled; Cancelled/Returned/Refunded status; ambiguous dates resolved by context | Review filled fields; exclude Cancelled from sales; isolate Returned/Refunded separately | 🟡 Medium |
| ✗ INVALID | 37 | 4.0% | Negative discount OR `ship_date` before `order_date` | Correct source data; do NOT use `calculated_sales` for these records in reporting | 🔴 High |

---

*Source: raw_orders.xlsx — 932 records assessed — 7 quality dimensions — Use the Calculated Data sheet for the analysis-ready dataset.*
