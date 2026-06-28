# abhigyansinha_2511978_part1_data_cleaning.
Business Data Cleaning, Validation &amp; Excel Reporting
**Problem summary: **
The raw dataset arrived in an uncleaned state that made it unreliable for any business analysis or reporting. Specifically:
Data Integrity Issues
The dataset contained 20 exact duplicate rows (identical across all 21 columns), and 12 order IDs that appeared more than once with conflicting values in sales and order_status — suggesting records were re-entered rather than updated. Without resolving these, any aggregation would double-count revenue or orders.
Inconsistent & Invalid Values
Discount values were a mix of valid (0–1), missing, and outright impossible — 16 records had negative discounts, which cannot exist. Text fields like region, segment, ship_mode, and category were stored in inconsistent casing (ALL CAPS, lowercase, Title Case) and contained leading, trailing, and internal double spaces, making GROUP BY operations unreliable. Three fields — region, ship_mode, and discount — had outright missing values in 22–26 rows each.
Date Chaos
Dates were stored in six different formats across order_date and ship_date with no standardisation. 377 of those dates were ambiguous — the day and month were both ≤ 12, making it impossible to determine whether 05/08/2024 meant 5 August or 8 May without additional context. On top of that, 21 records had a ship date that preceded the order date — a physical impossibility indicating data entry errors.
Business Logic Violations
146 cancelled orders and 164 returned orders were sitting in the same dataset as completed sales, meaning any revenue total pulled from the raw data would be materially wrong. 72 orders had a Refunded payment status with no separation from other records. No calculated columns existed — there was no profit_margin, no shipping_delay_days, no order_month — forcing anyone using the data to re-derive these every time, with no guarantee of consistency.
Financial Calculation Mismatches
For 46 records, the recorded sales value did not match the formula Qty × Unit Price × (1 − discount) by more than ₹1, with gaps as large as ₹16,803. This means the financial figures in the raw file cannot be trusted at face value without re-deriving them from source components.
The core problem in one sentence: The dataset could not be used for any sales reporting, regional analysis, or logistics review without first resolving structural errors, standardising formats, enforcing business rules, and deriving validated calculated fields — all of which this cleaning process addressed.
**Dataset Description**
What the Dataset Represents:
The dataset is a transactional order record from an Indian e-commerce or retail business. Each row represents a single order line — one product purchased by one customer in one transaction. It captures the full lifecycle of an order from placement through shipment, and tracks the financial outcome (sales, cost, profit) alongside the customer, product, and logistics context.
Column Inventory:
The 21 columns fall into six logical groups:
Order Identifiers
Column                   Description
order_id                 Unique identifier per order (format: ORD-YYYY-NNNNN)
order_date               Date the order was placed
ship_date                Date the order was dispacthed
Customer Details
Column                   Description
customer_id              Unique customer identifier
customer_name            Full name of the customer   
segment                  Customer Type- Consumer, Corporate,Home Office,Small Business
Geography
Column                   Description
region                   Broad region — North, South, East, West
sate                     Indian state of the customer  
city                     City of the customer
Product details
Column                   Description
category                 Top-level product category — Furniture, Office Supplies, Technology
sub_category             13 sub-categories (e.g. Chairs, Copiers, Binders, Phones)  
product name             Full Product Name
Logistics
Column                   Description
ship_mode                Shipping method — Standard Class, Second Class, First Class, Same Day
quantity                 Number of Units Ordered
Financials
Column                   Description
unit_price               Price per unit in ₹
discount                 Discount applied as a proportion (0 = no discount, 0.20 = 20% off) 
sales                    Recorded gross sales value in ₹
cost                     Cost of goods in ₹
profit                   Recorded profit in ₹
Order Status
Column                   Description
payment_status           Payment outcome — Paid, Pending, Refunded, Failed
order_status             Order outcome — Completed, Cancelled, Returned
Key Characteristics
Geography: Orders span all four regions of India (North, South, East, West) across multiple states and cities. The North region has the largest order volume.
Product mix: Three top-level categories — Technology, Furniture, and Office Supplies — each with 3–5 sub-categories. Copiers and Chairs are the highest-revenue sub-categories.
Customer segments: Four segments. Home Office has the highest profit margin (29.9%); Small Business the lowest (27.4%).
Order outcomes: 66.7% of orders are Completed, 17.6% Returned, and 15.7% Cancelled — meaning roughly one in three orders does not result in a completed sale.
Payment outcomes: 75.5% Paid, 9.3% Pending, 7.7% Refunded, 7.4% Failed.
Financials: Average unit price ≈ ₹2,281. Average order quantity ≈ 5 units. Completed sales total ₹60.5L across 622 orders. Average profit margin across completed orders is approximately 28–30%.
Time span: 23 months of data, from January 2024 through November 2025. Data thins out in the final months (October–November 2025 have very few records), suggesting the export was taken mid-cycle rather than at a natural period-end.
Relationship between columns: sales is theoretically derived as quantity × unit_price × (1 − discount). profit is theoretically sales − cost. However, 46 records show a discrepancy between the recorded sales and the formula result, indicating the discount was entered incorrectly at the time of recording for those orders.
**Tools Used**
Tool                    Role
Python / pandas         Data loading, inspection, cleaning, transformation, aggregation
Python / numpy          Numerical operations, conditional column derivation
Python / re             Date format classification, text normalisation
Python / dateutil       Multi-format date parsing
Python / openpyxl       Excel workbook construction and styling
Excel formulas          Live calculated columns, totals, rankings, flag logic
Excel AutoFilter        Filtering and sorting in pivot output sheets
**Cleaning Steps Performed**
Step 1 — Strip Leading and Trailing Spaces
Fields: customer_name, segment, region, state, city, category, sub_category, ship_mode, payment_status, order_status
Every text value was stripped of spaces at the start and end of the string. Values like "  North " and " Corporate" were reduced to their clean form.
Method: str.strip() in Python | =TRIM(A2) in Excel
Step 2 — Collapse Internal Double Spaces
Fields: segment, category, ship_mode
Values containing multiple consecutive spaces within the string — such as "Small  Business", "Office  Supplies", and "Standard  Class" — were reduced to a single space.
Method: re.sub(r'\s+', ' ', v) in Python | =SUBSTITUTE(A2,"  "," ") in Excel
Step 3 — Normalise Casing to Title Case
Fields: customer_name, segment, region, category, sub_category, ship_mode, payment_status, order_status
Values stored in ALL CAPS ("EAST", "FIRST CLASS"), all lowercase ("east", "home office"), or mixed case were standardised to Title Case across all eight text fields. This affected 106 cells.
Method: str.title() in Python | =PROPER(TRIM(A2)) in Excel
Step 4 — Fill Missing Region Values
Field: region | Records affected: 26
All 26 rows with a blank region field were filled with the value "Unknown". Inference from the state column was considered but rejected — the state-to-region mapping is business-specific and was not confirmed by the data owner.
Method: df['region'].fillna('Unknown') | =IF(ISBLANK(G2),"Unknown",G2)
Step 5 — Fill Missing Ship Mode Values
Field: ship_mode | Records affected: 22
All 22 rows with a blank ship_mode were filled with "Unknown". Statistical mode-imputation (filling with "Standard Class", the most common value) was rejected to avoid injecting unverified values.
Method: df['ship_mode'].fillna('Unknown') | =IF(ISBLANK(M2),"Unknown",M2)
Step 6 — Fill Missing Discount Values
Field: discount | Records affected: 26
Missing discount values were filled with 0 only where sales, cost, and profit were all present and greater than zero — indicating the order was transacted without a discount. All 26 missing rows met this condition. Where sales fields were absent or invalid, the value would have been left blank and flagged — but no such cases existed in this dataset.
Method: Conditional fillna(0.0) | =IF(AND(ISNUMBER(Q2),Q2>0,ISNUMBER(R2),R2>0,ISNUMBER(S2)),0,"INVALID")
Step 7 — Flag Negative Discount Values as Invalid
Field: discount | Records affected: 16
Sixteen records had a negative discount (e.g. -0.19, -0.23). A discount cannot be negative — these are data entry errors. The values were not corrected (the correct value is unknown) but were flagged as INVALID in the discount_flag column. The clean_discount column was left blank for these rows, and calculated_sales was set to "INVALID" to prevent these figures from contaminating any financial analysis.
Method: where disc < 0 → disc_clean = NaN | =IF(P2<0,"INVALID",P2)
Step 8 — Classify All Date Formats
Fields: order_date, ship_date | Records affected: All 932
Before parsing, every date value was classified into one of six format types using regular expression matching:
ISO (YYYY-MM-DD)
Named month (DD Mon YYYY, e.g. 21 Jul 2024)
Unambiguous slash (DD/MM/YYYY where day > 12, or MM/DD/YYYY where month > 12)
Unambiguous dash (same logic, dash-separated)
Ambiguous slash (both parts ≤ 12)
Ambiguous dash (both parts ≤ 12)
This classification was the prerequisite for all subsequent date parsing.
Step 9 — Parse Unambiguous Dates
Fields: order_date, ship_date | Records affected: 731 order_date + 756 ship_date
All date values with a determinable format were parsed directly into Python date objects using dateutil.parser with the appropriate dayfirst setting.
Step 10 — Resolve Ambiguous Dates by Context
Fields: order_date, ship_date | Records affected: 201 order_date + 176 ship_date
For the 377 date values where both the day and month components were ≤ 12 — making the format indeterminate — a context-aware resolution was applied. Both possible interpretations were computed (DD/MM and MM/DD), and the interpretation that produced a ship-to-order gap between 0 and 60 days was chosen. Where neither interpretation produced a valid window, MM/DD (the more common format in the dataset) was used as a fallback.
Assumption recorded: The 60-day window is assumed to be the maximum reasonable order lead time. This requires validation with the operations team.
Step 11 — Standardise All Dates to DD/MM/YYYY
Fields: order_date, ship_date | Records affected: All 932
All parsed dates — regardless of their original format — were written as Excel date serial numbers formatted as DD/MM/YYYY. The six original formats were collapsed into one consistent representation across both columns.
Method: Write as datetime.date object via openpyxl | =DATEVALUE(TRIM(B2)) then format as DD/MM/YYYY
Step 12 — Flag Ship Date Before Order Date
Field: ship_date vs order_date | Records affected: 21
After parsing, 21 records were found to have a ship_date earlier than their order_date. These are logistically impossible and indicate the two dates were likely swapped at the point of data entry. The records were not deleted or corrected — the root cause is unknown. They were flagged "INVALID" in the shipping_date_flag column and excluded from the shipping_delay_days calculation.
Method: if ship_date < order_date → flag = "Invalid" | =IF(AND(ISNUMBER(C2),ISNUMBER(B2)),IF(C2>=B2,C2-B2,"INVALID"),"N/A")
Step 13 — Remove Exact Duplicate Rows
Fields: All 21 columns | Records removed: 20
Twenty rows were identified as byte-for-byte identical to another row across all 21 columns. The first occurrence of each was retained; subsequent copies were deleted. This is the only step in the entire process where records were permanently removed. Zero unique information was lost.
Dataset after this step: 932 → 912 rows
Method: df.drop_duplicates(keep='first') | Excel: Data → Remove Duplicates (all columns selected)
Step 14 — Flag Conflicting Duplicate Order IDs
Field: order_id | Records affected: 25 rows across 12 order IDs
Twelve order IDs appeared more than once with different values in sales and/or order_status. Unlike exact duplicates, these cannot be safely removed — each row contains different information. Both rows were retained. The first occurrence was labelled "Conflict — Primary (Retained)" and the second "Conflict — Secondary (Flagged)" in the duplicate_flag column. Analyst investigation is required to determine which row is correct.
Method: Group by order_id → check if deduplicated group has more than one unique row | =SUMPRODUCT((A$2:A$932=A2)*(Q$2:Q$932<>Q2))>0
Step 15 — Add clean_discount Column
Source: discount | Records: 932
A new column was created containing the standardised discount value for each row:
Valid values (0–1): carried through unchanged
Missing values with valid sales fields: filled as 0
Negative or above-100% values: set to "INVALID"
This column is used as the sole input to all downstream financial calculations, preventing invalid raw discounts from affecting results.
Formula: =IF(ISNUMBER(P2),IF(OR(P2<0,P2>1),"INVALID",P2),IF(AND(ISNUMBER(Q2),Q2>0,ISNUMBER(R2),R2>0,ISNUMBER(S2)),0,"INVALID"))
Step 16 — Add calculated_sales Column
Source: quantity, unit_price, clean_discount | Records: 916 valid, 16 INVALID
calculated_sales = Qty × Unit Price × (1 − clean_discount), rounded to 2 decimal places. Where clean_discount is "INVALID", the result is also "INVALID". This column replaces reliance on the recorded sales field for financial analysis, as 46 records showed discrepancies between the two.
Formula: =IF(V2="INVALID","INVALID",ROUND(N2*O2*(1-V2),2))
Step 17 — Add calculated_profit Column
Source: calculated_sales, cost | Records: 916 valid, 16 INVALID
calculated_profit = calculated_sales − cost, rounded to 2 decimal places. "INVALID" propagates from calculated_sales.
Formula: =IF(W2="INVALID","INVALID",ROUND(W2-R2,2))
Step 18 — Add profit_margin Column
Source: calculated_profit, calculated_sales | Records: 916 valid
profit_margin = calculated_profit ÷ calculated_sales, stored as a decimal (e.g. 0.29 = 29%). Returns "N/A" where calculated_sales is zero or "INVALID".
Formula: =IF(OR(W2="INVALID",W2=0),"N/A",ROUND(X2/W2,4))
Step 19 — Add shipping_delay_days Column
Source: order_date, ship_date | Records: 911 valid, 21 INVALID
shipping_delay_days = ship_date − order_date as an integer number of days. Returns "INVALID" where ship date precedes order date, and "N/A" where either date is missing or unparseable.
Formula: =IF(AND(ISNUMBER(B2),ISNUMBER(C2)),IF(C2>=B2,C2-B2,"INVALID"),"N/A")
Step 20 — Add order_month and order_year Columns
Source: order_date | Records: 932
Two time-dimension columns were derived to support trend analysis and grouping:
order_month: formatted as "Jan 2024" — =IF(ISNUMBER(B2),TEXT(B2,"MMM-YYYY"),"N/A")
order_year: integer year — =IF(ISNUMBER(B2),YEAR(B2),"N/A")
Step 21 — Assign data_quality_flag Column
Source: All columns | Records: 932
A three-level quality flag was assigned to every row based on all prior cleaning steps:
CLEAN (547 rows) — All fields valid, dates chronological, no anomalous status
WARNING (348 rows) — Any filled missing value, or Cancelled / Returned / Refunded / Failed status, or an ambiguous date resolved by context assumption
INVALID (37 rows) — Negative discount or ship date before order date
Formula: Nested IF across clean_discount, shipping_date_flag, discount, ship_mode, region, order_status, and payment_status columns.
**Business Rules Applied**
Nine business rules were defined and applied to the dataset. Each rule reflects a decision about how the data should behave from a business perspective — distinct from pure data quality checks, which are about what the data is.
Rule 1 — Missing Region: Fill as "Unknown"
Field: region
Condition: If the region field is blank
Action: Fill with the value "Unknown". Do not delete the record. Flag the row as WARNING in data_quality_flag.
Why: A blank region prevents the record from contributing to regional analysis, but the order itself is valid. Deleting the record would lose a real transaction. Filling with "Unknown" preserves the record and makes the gap visible rather than hiding it.
Constraint: The value "Unknown" must never be included in regional sales totals or comparisons without being explicitly called out. Downstream analysts must filter or annotate it separately.
Records affected: 26 rows
Rule 2 — Missing Ship Mode: Fill as "Unknown"
Field: ship_mode
Condition: If the ship_mode field is blank
Action: Fill with "Unknown". Do not delete the record. Do not impute with the most common value (Standard Class). Flag as WARNING.
Why: Statistical imputation — filling the mode with the most frequent category — would inject an assumption into the data that cannot be verified. "Unknown" is more honest and ensures these 22 records do not distort shipping mode analysis.
Constraint: "Unknown" ship mode records must be excluded from any logistics cost or shipping performance analysis.
Records affected: 22 rows
Rule 3 — Missing Discount: Fill as Zero (Conditional)
Field: discount
Condition: Discount is blank AND sales > 0 AND cost > 0 AND profit is present
Action: Fill with 0 (no discount applied). If the sales fields are not all valid, leave blank and flag INVALID.
Why: A missing discount on an otherwise complete order most plausibly means no discount was applied. The condition ensures the fill is only made when there is enough surrounding evidence to support it — a full set of valid financial figures implies the transaction completed normally.
Constraint: If even one of sales, cost, or profit is missing or zero, the fill is not made. The record is flagged INVALID instead.
Records affected: 26 rows filled as 0 (all 26 met the condition — 0 were flagged INVALID under this rule)
Rule 4 — Negative Discount: Flag as Invalid
Field: discount
Condition: Discount value is less than zero
Action: Flag discount_flag = "Invalid — Negative discount". Set clean_discount = INVALID. Do not use in calculated_sales or any financial calculation. Do not delete the record.
Why: A discount cannot be negative — it has no valid business meaning. These 16 values are data entry errors. However, because the correct value is unknown (it could be 0, a positive discount, or something else entirely), no correction is made. The record is retained but excluded from all financial outputs.
Constraint: calculated_sales, calculated_profit, and profit_margin are all set to "INVALID" for these rows and must not appear in any revenue or margin summary.
Records affected: 16 rows
Rule 5 — Discount Ceiling: Flag Above 100% as Invalid
Field: discount
Condition: Discount value is greater than 1.0 (i.e. greater than 100%)
Action: Flag as INVALID. Exclude from calculated columns. Do not delete.
Why: A discount greater than 100% means the customer is being paid to take the product — which is not a valid commercial transaction in this context. Like negative discounts, these are data entry errors where the correct value is unknown.
Outcome in this dataset: Zero violations found. The rule was documented and the check runs on every load so it will catch future occurrences automatically.
Records affected: 0 rows in this dataset
Rule 6 — Cancelled Orders: Exclude from Completed Sales
Field: order_status
Condition: order_status = "Cancelled"
Action: Flag row with cancelled_flag = "Cancelled — Excluded from Completed Sales". Exclude from all completed sales totals, revenue figures, and profit calculations. Show in a dedicated Cancelled summary table only.
Why: A cancelled order was never fulfilled. Including it in revenue figures would overstate sales. The 146 cancelled orders represent ₹14,58,700.65 in gross sales that must not appear in the completed sales total.
Constraint: Cancelled orders are retained in the full dataset but must never contribute to the Completed Sales figure. Note that 39 cancelled orders have payment_status = "Paid" — these may require refund processing and should be reviewed separately.
Records affected: 146 rows
Rule 7 — Returned Orders: Summarise Separately
Field: order_status
Condition: order_status = "Returned"
Action: Isolate in a dedicated Returns summary. Do not include in the Completed Sales total. Do not delete.
Why: A returned order was initially fulfilled but subsequently reversed. It represents a different business event from a completed sale and needs its own reporting line to understand return rates, returned revenue, and the financial impact of returns.
Constraint: Returned orders (₹16,32,404.84 gross sales across 164 records) must appear only in the Returns summary, never in the Completed Sales total.
Records affected: 164 rows
Rule 8 — Refunded Payments: Summarise Separately
Field: payment_status
Condition: payment_status = "Refunded"
Action: Isolate in a dedicated Refunds summary separate from all other summaries — including the Returns summary. Do not delete.
Why: Refunded payments cut across order status categories — 38 come from Cancelled orders and 34 from Returned orders. This means a simple filter on order_status would not capture all refunds, and a simple filter on payment_status = "Refunded" would. The refund pool (₹6,77,707.88 across 72 records) must be reported as a distinct line for cash flow and accounts reconciliation purposes.
Constraint: Refunded records must not be merged into either the Cancelled total or the Returned total. They require their own table.
Records affected: 72 rows (38 Cancelled + 34 Returned)
Rule 9 — Ship Date Before Order Date: Flag as Invalid
Fields: ship_date, order_date
Condition: ship_date < order_date after both dates have been parsed and standardised
Action: Flag shipping_date_flag = "Invalid — Ship Date Before Order Date". Set shipping_delay_days = "INVALID". Retain the record. Do not correct the dates (the correct values are unknown).
Why: It is physically impossible for an order to be shipped before it is placed. The 21 records where this occurs have dates that were almost certainly swapped at the point of data entry. Correcting them by swapping would be an assumption — the correct action is to flag them and refer back to the source system.
Constraint: These 21 records must be excluded from all shipping delay analysis and any logistics performance metrics. They may still contribute to sales summaries if their order status and discount are valid.
Records affected: 21 rows
**Summary of data quality issues found**
1. Missing Values
Three fields had blank entries — region (26 rows), ship_mode (22 rows), and discount (26 rows). All were filled using business rules rather than deleted.
2. Text Casing & Whitespace
106 cells across 8 text fields had inconsistent casing (ALL CAPS, lowercase, mixed) and extra spaces — including leading/trailing spaces and internal double spaces like "Small  Business".
3. Invalid Discount Values
16 records had a negative discount — a value that has no valid business meaning. 0 records exceeded 100%. The 16 negative values were flagged Invalid and excluded from all financial calculations.
4. Mixed Date Formats
6 different date formats were found across order_date and ship_date — including ISO, DD Mon YYYY, slash-separated, and dash-separated. 377 dates were ambiguous (both day and month ≤ 12) and could not be determined from the value alone.
5. Ship Date Before Order Date
21 records had a ship date earlier than the order date — physically impossible and indicating the two dates were swapped at entry.
6. Duplicate Records
20 rows were exact byte-for-byte copies of another row and carried zero new information. A further 12 order IDs appeared twice with conflicting sales and order_status values — indicating an amendment or return was logged as a new record rather than updating the original.
7. Order Status Issues
310 records — 146 Cancelled and 164 Returned — were sitting in the same dataset as completed sales, which would overstate revenue if included in any total. An additional 72 orders had a Refunded payment status requiring a separate summary.
8. Sales & Profit Mismatch
In 46 records, the recorded sales figure did not match the formula Qty × Unit Price × (1 − discount) by more than ₹1 — with gaps up to ₹16,803. Profit followed in 47 records. The likely cause is a wrong discount entered at the time of the transaction.
9. Final Record Quality
After all cleaning and rules were applied, 548 records (58.8%) were CLEAN, 347 (37.2%) carried a WARNING (filled values or non-completed status), and 37 (4.0%) were INVALID (negative discount or impossible ship date).
**Summary of final pivot reports**
Report 1 — Sales & Performance
Table 1 · Sales & Profit by Region
South leads on total sales (₹16.6L) but East has the best margin (29.9%). North has the lowest sales volume. The four known regions perform within a tight 2.7 percentage point margin band — no single region dramatically outperforms another.
Table 2 · Sales & Profit by Category & Sub-category
Technology is the highest-revenue category, driven by Copiers (₹6.8L) and Phones (₹6.1L). Within Furniture, Tables deliver the best margin (30.3%) despite not being the top seller. Office Supplies has the widest spread — Binders margin (31.6%) is nearly 7 points above Storage (24.6%), the weakest sub-category in the entire dataset.
Report 2 — Operations & Trend
Table 3 · Order Count by Ship Mode
All four ship modes handle roughly equal volumes (211–245 orders each). Standard Class is the most used. Completion rates are consistent across modes — hovering around 65–70% — suggesting ship mode choice does not materially affect whether an order completes.
Table 4 · Profit Margin by Segment
Home Office leads with a 29.9% margin, followed closely by Corporate (29.3%) and Consumer (28.3%). Small Business trails at 27.4%. The gap between best and worst is only 2.5 percentage points — margins are broadly consistent across all customer types.
Table 5 · Problematic Orders by Region
North has the highest count of Cancelled and Returned orders (49 Cancelled, 37 Returned — 86 combined) despite not being the top-sales region, giving it the worst problem rate. South and West are broadly even. East has the fewest cancellations (30) but the most returns (46).
Table 6 · Monthly Sales Trend
2024 full-year completed sales totalled approximately ₹29.5L across 328 orders. 2025 (Jan–Nov, partial) reached approximately ₹27.7L across 276 orders, suggesting broadly similar annual performance. The strongest months were February 2025 (₹4.2L), June 2024 (₹3.6L), and May 2024 (₹3.4L). July 2024 and September 2024 were the softest months. November 2025 has only one record — the data was likely exported mid-month.
**Key Business Insights **
1. One in Three Orders Does Not Complete
Of 932 total orders only 622 (66.7%) reached Completed status. The remaining 33.3% split between Cancelled (15.7%) and Returned (17.6%). This is a significant fulfilment problem — for every three orders placed, one either never ships or comes back. The business is generating and processing a large volume of transactions that produce no lasting revenue.
2. Revenue Is Being Overstated in Raw Reporting
The raw dataset includes ₹14,58,700 in cancelled sales and ₹16,32,405 in returned sales sitting alongside completed revenue. Any report built directly from the raw file without filtering would show total sales of approximately ₹91L — nearly 50% higher than the true completed figure of ₹60.5L. This has likely been distorting any dashboard or summary built before this cleaning exercise.
3. North Region Has a Cancellation Problem
North has the highest cancellation count (49) of any region despite having the lowest completed sales volume. This is a disproportionate rate — North is both the smallest revenue contributor and the most cancellation-prone. This warrants investigation into whether the issue is demand-side (customers changing their minds), supply-side (stock unavailability), or operational (fulfilment delays triggering cancellations).
4. ₹6.8L in Sales Cannot Be Trusted at Face Value
46 records have a recorded sales value that does not match Qty × Unit Price × (1 − discount) by more than ₹1 — with gaps as large as ₹16,803 on a single order. The root cause is a wrong discount entered at point of sale. Any margin or revenue analysis built on the raw sales column for these records will be incorrect. The calculated_sales column resolves this, but the source system entries need correcting.
5. Technology Is the Revenue Engine, but Furniture Hides the Best Margins
Technology generates the most completed sales revenue, led by Copiers and Phones. However, within Furniture, Tables deliver a 30.3% margin — one of the highest of any sub-category across the entire product range. Machines (Technology) have the single best margin at 34.8%. Office Supplies Storage is the weakest performer at 24.6% — nearly 10 points below Machines — and may warrant a pricing or cost review.
6. Small Business Segment Is the Least Profitable Customer Type
Home Office customers deliver the highest profit margin at 29.9%, while Small Business sits at 27.4% — a 2.5 point gap that compounds across 158 completed orders. Given that Small Business also has the second-highest order count, improving margin here through pricing, discount discipline, or product mix could have a material impact on overall profitability.
7. Discounts Are Being Applied Incorrectly in 16 Cases
Sixteen orders were submitted with a negative discount — meaning the system accepted a value that makes no commercial sense. This points to a validation gap in the order entry system. If negative discounts were processed as recorded, customers may have been overcharged. If the system corrected them silently, the data trail is unreliable. Either way, an input validation rule should be added at the source.
8. 72 Refunded Orders Totalling ₹6.8L Have No Dedicated Tracking
Refunded orders span both Cancelled (38) and Returned (34) statuses, meaning they are invisible in standard order status reporting. Without a dedicated refund view, finance has no reliable way to track cash leaving the business through refunds. The ₹6,77,708 in refunded sales represents real money that has been returned to customers and must be reconciled against revenue.
9. 69 Failed-Payment Orders Are Currently Counted as Revenue
Orders with payment_status = "Failed" were fulfilled but never paid for. These 69 records are included in completed sales by default because their order_status = "Completed". Depending on the accounting policy, this could mean revenue is being recognised on transactions where cash was never collected — a potential compliance issue and certainly a collections risk.
10. Shipping Performance Is Consistent but 21 Records Flag a Process Gap
All four ship modes deliver broadly similar completion rates and average delay times. However, 21 records where the ship date precedes the order date point to a data entry or system integration problem — dates are either being entered in the wrong fields, or the logistics system is logging events out of sequence. Left unaddressed, this corrupts all shipping delay analysis and SLA reporting.
11. February Is Consistently the Strongest Month
February appears as a peak month in both years — ₹2.9L in February 2024 and ₹4.2L in February 2025, with February 2025 being the single strongest month in the entire dataset. This may reflect a seasonal demand pattern (post-January budget releases, procurement cycles, or a recurring promotion) that the business could plan inventory and staffing around more **deliberately.
12. Profit Margins Are Healthy but Narrow Across the Board
Across all completed orders, profit margins range from approximately 24.6% (Office Supplies Storage) to 34.8% (Technology Machines) — a 10-point spread. The overall blended margin sits around 28–30%. This is a relatively healthy range, but the consistency also means there is limited buffer. A 2–3 point margin erosion from increased returns, discount misuse, or cost increases would be immediately material.
**Assumptions and Limitations**
Assumptions
Six assumptions were made during the cleaning process where the data alone was ambiguous and the business rule could not be confirmed from the data itself. Each carries a risk if wrong and should be validated with the relevant team.
Assumption 1 — Missing Region Filled as "Unknown", Not Inferred from State
What was assumed: The region field was filled with "Unknown" for 26 blank rows rather than being derived from the state column.
Why: A state-to-region mapping could theoretically infer the region (e.g. Gujarat → West, Tamil Nadu → South). However, this mapping is business-specific — the same state could belong to a different sales region depending on how the business has drawn its territory boundaries. Applying an unverified map could introduce incorrect region assignments that look correct but are wrong.
Risk if wrong: 26 records remain unassigned to a region, slightly understating each region's true order count and sales. Low operational risk but medium analytical risk if region-level decisions are being made from this data.
Validate with: Data Owner or Geography / Territory Team
Assumption 2 — Missing Ship Mode Filled as "Unknown", Not Mode-Imputed
What was assumed: The 22 blank ship_mode values were filled with "Unknown" rather than with "Standard Class" — the most frequent value in the dataset.
Why: Filling with the statistical mode (most common value) is a common technique but introduces a manufactured value that cannot be verified. If even one of the 22 orders was actually shipped by Same Day or First Class, the imputed value would be wrong and would distort shipping cost and SLA analysis.
Risk if wrong: 22 records are excluded from shipping mode analysis. If they all genuinely were Standard Class, the Standard Class totals are slightly understated. Low risk.
Validate with: Operations or Logistics Team — check carrier records for the 22 affected order IDs.
Assumption 3 — Ambiguous Dates Resolved Using a 0–60 Day Ship Window
What was assumed: For 377 date values where both the day and month components were ≤ 12 — making the format indeterminate (e.g. 05/08/2024 could be 5 August or 8 May) — the interpretation was chosen as whichever produced a ship-to-order gap between 0 and 60 days.
Why: Without knowing the locale setting of the system that generated the data, both interpretations are equally plausible from the value alone. The 0–60 day window was used as a proxy for a reasonable order lead time. Where neither interpretation produced a valid window, MM/DD was used as a fallback (the more common format in the rest of the dataset).
Risk if wrong: Up to 377 dates may be incorrectly interpreted — meaning the wrong month and year are recorded for those orders, corrupting monthly trend analysis, shipping delay calculations, and order_month / order_year derived columns. This is the highest-risk assumption in the dataset.
Validate with: Operations Team — confirm the maximum order lead time policy. IT Team — confirm the date locale setting of the source system or ERP.
Assumption 4 — Cancelled Orders Are Business Events, Not Data Errors
What was assumed: The 146 Cancelled orders were treated as legitimate business records — real orders that were placed and subsequently cancelled — rather than as erroneous entries that should be deleted.
Why: All 146 records have valid, internally consistent data across all fields. There is no signal that they are duplicates, test entries, or system errors. Cancellation is a normal order lifecycle outcome.
Risk if wrong: If some cancellations represent partially fulfilled orders (where goods were dispatched before cancellation), excluding them entirely from sales could understate true revenue. Additionally, 39 of the 146 cancelled orders have payment_status = "Paid" — meaning payment was collected before cancellation. These may require refund processing that has not yet occurred.
Validate with: Finance and Revenue Team — confirm the revenue recognition policy for cancelled orders.
Assumption 5 — calculated_sales = Qty × Unit Price × (1 − discount)
What was assumed: The correct formula for gross sales is quantity × unit_price × (1 − discount). This formula was applied as the standard to validate and replace the recorded sales field where discrepancies exist.
Why: The formula matched 870 of 932 records within ±₹1 rounding tolerance — an 93.4% match rate — suggesting it is correct for the majority of the dataset. The 46 mismatched records are most plausibly explained by a wrong discount value being entered at point of sale.
Risk if wrong: If the sales figure includes additional charges not captured in these three columns — such as tax, shipping fees, handling charges, or bundle pricing — then calculated_sales will consistently understate true sales. Financial totals using calculated_sales would then be systematically lower than reality.
Validate with: Finance Team — confirm the exact sales calculation formula used in the source system. Confirm whether sales is pre-tax or post-tax, and whether it includes any additional line items beyond product price and discount.
Assumption 6 — profit = calculated_sales − cost with No Other Deductions
What was assumed: Profit was computed purely as calculated_sales − cost. No adjustments were made for tax, overhead allocation, return processing fees, refund transaction costs, or any other deductions.
Why: The dataset contains only two financial inputs that can be subtracted — sales (or calculated_sales) and cost. There is no column for tax, overheads, or other charges. In the absence of those fields, the simplest valid formula was used.
Risk if wrong: If the business calculates profit after additional deductions not present in this dataset, then the profit_margin figures (approximately 28–30% across completed orders) are overstated. True net margin could be materially lower.
Validate with: Finance Team — confirm the P&L structure and whether cost represents full landed cost or only direct product cost.

Limitations
Five limitations constrain what the cleaning process could achieve, regardless of the effort applied. These are structural constraints rather than oversights.
Limitation 1 — No Access to the Source System for Ground Truth
What the limitation is: The entire cleaning process was applied to the exported flat file only. There was no connection to the originating database, ERP, CRM, or order management system. Every decision was made based solely on the data visible in the file.
Impact: Any value that is internally consistent but factually wrong cannot be detected. For example, a sales figure of ₹45,000 for a 2-unit order at ₹20,000 per unit with a 10% discount would pass every validation check — but if the actual transaction was for 3 units, the error is invisible. Systematic pricing errors, wrong customer assignments, or duplicate orders from different entry points would all escape detection.
How serious: High. This is the most fundamental constraint on any flat-file cleaning exercise.
Mitigation: Connect the dataset to the source system for reconciliation. Cross-check order_id, customer_id, and sales figures against the live database. Treat this cleaned file as a best-effort improvement, not a guaranteed accurate dataset.
Limitation 2 — Ambiguous Date Interpretation Cannot Be Fully Verified
What the limitation is: 377 dates were resolved using a contextual heuristic (the 0–60 day ship window). This cannot be independently confirmed as correct without knowing the locale of the system that generated the data.
Impact: Up to 377 order records may have incorrect dates — affecting order_month, order_year, monthly trend analysis, and shipping_delay_days. In the worst case, orders are being attributed to the wrong month or even the wrong quarter. For a 23-month dataset, misattributing even 50 orders could visibly distort monthly trend lines.
How serious: High. Monthly and quarterly analysis should be treated with caution until the date formats are confirmed.
Mitigation: Ask the IT or data engineering team for the locale setting of the source system. Alternatively, pull a sample of orders and cross-check their dates against order confirmation emails or system logs.
Limitation 3 — 12 Conflicting Order IDs Remain Unresolved
What the limitation is: Twelve order IDs appear twice with different values. Both rows were retained and flagged, but the correct row has not been identified. The root cause — whether it is a system re-entry, an amendment, a return event, or a data integration error — is unknown.
Impact: These 25 rows (12 primary + 13 secondary) cannot be safely included in analysis. Including both rows risks double-counting; including only the primary risks using the wrong version. Excluding them entirely removes valid transactions. Until resolved, they represent a ₹X revenue figure that cannot be reliably attributed.
How serious: High for accuracy. Medium for scale — 25 rows is 2.7% of the dataset.
Mitigation: Each of the 12 order IDs must be investigated in the source system individually. Determine what business event created the second row — if it is a legitimate amendment, the primary should be archived and the secondary promoted; if it is a data entry error, the secondary should be deleted.
Limitation 4 — Sales Formula Mismatch in 46 Records Not Corrected at Source
What the limitation is: For 46 records, the recorded sales value differs from Qty × Unit Price × (1 − discount) by more than ₹1. The calculated_sales column provides a corrected figure, but the source system still holds the wrong value. The root cause (wrong discount at entry) was assumed, not confirmed.
Impact: Any report or system that pulls directly from the source file — rather than using calculated_sales — will continue to use the incorrect figures. The total discrepancy across the 46 records averages ₹1,992 per order, with a maximum single-order gap of ₹16,803. If these are systematically in one direction, the cumulative revenue impact could be material.
How serious: Medium. The corrected figures are available in calculated_sales, but the problem persists in the source system.
Mitigation: Use calculated_sales for all financial reporting from this cleaned file. Separately, raise the 46 affected order IDs with the Finance or Sales Operations team for correction in the source system. Add a discount validation rule at point of entry to prevent recurrence.
Limitation 5 — Failed Payment Orders Included in Sales by Default
What the limitation is: 69 orders with payment_status = "Failed" have order_status = "Completed" and are therefore included in completed sales figures. Whether these represent collectible receivables, write-offs, or system errors depends on the accounting policy — information not available in this dataset.
Impact: If failed-payment orders are uncollectable, completed sales are overstated by whatever those 69 orders contributed. If some have since been collected through follow-up, including them is correct. Without knowing the resolution status of each, the right treatment cannot be determined from the data alone.
How serious: Medium for revenue accuracy. Potentially high if the business is recognising revenue on transactions where cash was never received — a compliance risk depending on the applicable accounting standard.
Mitigation: Confirm the accounting treatment of failed-payment orders with the Finance Team. If they are uncollectable, add a filter to exclude payment_status = "Failed" from all revenue reports. If they are in collections, add a separate receivables tracking column.
**Screenshots**
<img width="942" height="362" alt="raw_data_preview" src="https://github.com/user-attachments/assets/1b00d92d-43b2-4da4-9cd5-c56c2390e1b4" />
<img width="928" height="336" alt="cleaned_data_preview" src="https://github.com/user-attachments/assets/cbae1d08-423a-43e3-8ce1-657e7aba1eb6" />
<img width="861" height="319" alt="pivot_summary_1" src="https://github.com/user-attachments/assets/a9bf7e30-3c9d-4fff-a7d4-a54ddbd7e652" />
<img width="866" height="320" alt="pivot_summary_2" src="https://github.com/user-attachments/assets/5880b32b-4aec-4102-9096-5d24b4d3adfb" />
