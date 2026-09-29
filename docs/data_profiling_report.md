# Data Profiling Report — Olist Brazilian E-Commerce

Source: [Kaggle, olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

Profiled by: Abhishek Patra

Last updated: 2026-09-29

## 1.Inventory

| File | Rows | Cols | Null Cols | Primary Key | PK Valid? |
|------|------|------|-----------|-------------|-----------|
| olist_sellers_dataset.csv | 3095 | 4 | 0 | seller_id | Yes - 3095 Distinct and not null values |
| olist_customers_dataset.csv | 99441 | 5 | 0 | customer_id | Yes — 99,441 distinct, but order-scoped (see F5) |
| olist_orders_dataset.csv | 99441 | 8 | 3 | order_id | Yes — 99,441 distinct |
| olist_order_items_dataset.csv | 112650 | 7 | 0 | order_id,order_item_id | Yes — 112,650 distinct pairs, no nulls |


### Column cardinality — sellers
| Column | Distinct | Note |
|--------|----------|------|
| seller_id | 3095 | = Row count |
| seller_zip_code_prefix | 2246 | not unique per seller |
| seller_city | 611 | includes variant spellings (see F4) |
| seller_state | 23 | distinct codes present; includes DF (Federal District) | 

### Column cardinality — customers
| Column | Distinct | Note |
|--------|----------|------|
| customer_id | 99441 | = Row count(See F5) |
| customer_unique_id | 96096 | 96,096 distinct across 99,441 rows — repeats (see F6) |
| customer_zip_code_prefix | 14994 | not unique per customer |
| customer_city | 4119 | not checked for variant spellings (see F4) |
| customer_state | 27 | more codes than seller_state (23) |

### Column cardinality — orders
| Column | Distinct | Note |
|--------|----------|------|
| order_id | 99441 | Unique order id |
| customer_id | 99441 | Same number of rows as of customers table |
| order_status | 8 | approved,canceled,created,delivered,invoiced,processing,shipped,unavailable |
| order_purchase_timestamp | 98875 | 566 rows share timestamp with another order |
| order_approved_at | 90733 | 160 Null rows |
| order_delivered_carrier_date | 81018 | 1783 Null rows |
| order_delivered_customer_date | 95664 | 2965 Null rows |
| order_estimated_delivery_date| 459 | Date only no time component like other 4 |

### Column cardinality — items
| Column | Distinct | Note |
|--------|----------|------|
| order_id | 98666 | repeats — up to 21 rows per order; 775 orders have no items (see F15) |
| order_item_id | 21 | position within an order; numbered 1, 2, 3… up to the order's row count, no gaps; max 21 (see F14) |
| product_id | 32951 | not yet checked against the products table (see Q1) |
| seller_id | 3095 | same 3,095 sellers as the sellers table, both directions (see F17) |
| shipping_limit_date | 93318 | max 2020-04-09, later than every date in the orders table — not investigated yet (see Q4) |
| price | 5968 | min 0.85, max 6,735.00 — no zero or negative prices |
| freight_value | 6999 | min 0.00 — 383 rows in 339 orders have zero freight (see Q3) |

## 2.Findings

### F1 - seller_zip_code_prefix is not unique per seller

**Table :** olist_sellers_dataset.csv.

**Checked :** Distinct 'seller_zip_code_prefix' vs row count.

**Found :** 2,246 distinct values across 3,095 sellers. The most common prefix covers 49 sellers.

**Means :** Multiple sellers share same zip code,so it identifies 'location' and not a seller.

**Action :** Treat it as descriptive attribute,not as a key.

### F2 - Seller base is heavy at SP state

**Table :** olist_sellers_dataset.csv.

**Checked :** Counted sellers per state (value_counts on seller_state).

**Found :** SP holds 1,849 of 3,095 sellers (60%); top 3 (SP, PR, MG) hold 2,442 (79%).

**Means :** Large number of sellers are in SP compared to other states.

**Action :** None for the model. Recorded as context: 20 state codes have fewer than 50 sellers each.

### F3 - Zip is parsed as int64,stripping leads zero

**Table :** olist_sellers_dataset.csv.

**Checked :** Counted rows where prefix is shorter than 5 characters.

**Found :** 1,027 rows (33%) hold 4-character values. Lengths are only 4 or 5, so every 4-character value is a 5-digit prefix whose leading zero was dropped on read. Customers was read with dtype=str from the start, so the equivalent count there is assumed, not measured.

**Means :** The source CSV is correct — it contains "04195". The zero was destroyed by pandas inferring int64 on read. This is a defect in our ingestion, not in the source data.

**Action :** Read zip prefix columns with dtype=str at ingestion, in sellers and customers. Add a DQ test asserting length == 5.

### F4 - seller_city contains variant spellings of the same city

**Table :** olist_sellers_dataset.csv.

**Checked :** Filtered seller_city for 'do rio preto'.

**Found :** 'sao jose do rio preto' (33) and 's jose do rio preto' (1) — the same city under two values.

**Means :** Any group-by on raw city splits this city in two. Other abbreviations likely exist across the 611 values.

**Action :** Do not use raw city as a grouping key. Standardise in Silver against a municipality reference list. Extent of the problem across all 611 values not yet measured.


### F5 - customer_id is unique per row but does not identify a customer

**Table :** olist_customers_dataset.csv.

**Checked :** Compared distinct customer_id and customer_unique_id against total row.

**Found :** 99,441 distinct customer_id across 99,441 rows; 96,096 distinct customer_unique_id.

**Means :** Customer ID column is order ID as per the description of the kaggle data.

**Action :** Do not use customer_id to count or identify customers.

### F6 - repeated customer_unique_id

**Table :** olist_customers_dataset.csv.

**Checked :**  Counted customer_unique_id values appearing on more than one row.

**Found :** 2,997 of 96,096 customer_unique_id values (3.1%) appear on more than one row.

**Means :** customer_unique_id repeats within this table while customer_id does not. It is one row per order verified. (See F8)

**Action :** Group on customer_unique_id, not customer_id, for any customer-level count or aggregation — the two differ by 3,345 (99,441 vs 96,096).

### F7 - customer_state covers more state codes than seller_state

**Table :** olist_sellers_dataset.csv, olist_customers_dataset.csv.

**Checked :** Compared the set of distinct seller_state values against the set of distinct customer_state values, in both directions.

**Found :** 23 distinct codes in seller_state, 27 in customer_state. Codes appear in customer_state but not seller_state: ['AL', 'AP', 'RR', 'TO']. Appear in seller_state but not customer_state: [].

**Means :** The two columns do not cover the same set of state codes.

**Action :** Build the state/geography dimension from the union of seller_state and customer_state. Add a DQ test asserting every state value in both source tables resolves to a row in that dimension.

### F8 - One row represents one order

**Table :** olist_orders_dataset.csv

**Checked :** Compared distinct order_id and distinct customer_id against row count.

**Found :** 99,441 rows, 99,441 distinct order_id, 99,441 distinct customer_id. The customer_id sets in orders and customers match exactly in both directions — no orphans either way.

**Means :** One row per order. customer_id is unique per order here, confirming from data what the customers table suggested — it identifies an order, not a person.

**Action :** Use order_id as the primary key. Carry customer_id for traceability back to the source, not as a customer reference — the customer key is customer_unique_id.

### F9 - 160 orders have no approval timestamp

**Table :** olist_orders_dataset.csv

**Checked :** Counted nulls in order_approved_at, then broke those rows down by order_status.

**Found :** 160 rows have a null order_approved_at. Status breakdown: canceled 141, delivered 14, created 5.

**Means :** Per the Kaggle data dictionary, order_approved_at records payment approval. A null means the order never reached that stage. Cancellation (141) and created (5) account for 146 of these, where a null is expected. The remaining 14 have status 'delivered' — an order cannot be delivered without its payment being approved. All 14 were purchased in a narrow window — twelve on 17-19 Feb 2017, two on 19 Jan 2017 — suggesting a one-off processing failure rather than a recurring problem. All 14 have complete carrier and customer delivery timestamps, so the orders were fulfilled normally; only the approval record is missing.

**Action :** Treat null order_approved_at as "not approved", not as missing data — do not drop these rows in Silver. Add a DQ test asserting every order with status 'delivered' has a non-null order_approved_at; the 14 rows failing it today are a known defect.

### F10 - 1,783 orders have no carrier handover timestamp

**Table :** olist_orders_dataset.csv

**Checked :** Counted nulls in order_delivered_carrier_date, then broke those rows down by order_status.

**Found :** 1,783 rows have a null order_delivered_carrier_date. Status breakdown: unavailable 609, canceled 550, invoiced 314, processing 301, created 5, approved 2, delivered 2. The breakdown sums to 1,783.

**Means :** 1,781 of these sit at statuses before dispatch, where a null is expected — the order never reached the carrier. The 2 rows with status 'delivered' are inconsistent: an order cannot be delivered without being handed to a carrier.

**Action :** Treat null carrier date as expected for pre-dispatch statuses. Add a DQ test asserting every order with status 'delivered' has a non-null order_delivered_carrier_date — the 2 rows failing it today are a known defect and must be handled explicitly in Silver, not silently dropped.

### F11 - 2,965 orders have no customer delivery timestamp

**Table :** olist_orders_dataset.csv

**Checked :** Counted nulls in order_delivered_customer_date, then broke those rows down by order_status.

**Found :** 2,965 rows (3.0%) have a null order_delivered_customer_date. Status breakdown: shipped 1107, canceled 619, unavailable 609, invoiced 314, processing 301, delivered 8, created 5, approved 2. The breakdown sums to 2,965.

**Means :** The pre-delivery statuses are expected. The 8 rows with status 'delivered' are inconsistent — the status claims delivery with no timestamp recording it. These 8, offset by the 6 cancelled orders that do carry a delivery timestamp (F12), account for the difference between 2,965 nulls and the 2,963 orders whose status is not 'delivered'.

**Action :** Add a DQ test asserting every order with status 'delivered' has a non-null order_delivered_customer_date. Exclude these rows from delivery-time calculations rather than treating them as zero-duration.

### F12 - 6 cancelled orders have delivery timestamps

**Table :** olist_orders_dataset.csv

**Checked :** Filtered for order_status = 'canceled' with a non-null order_delivered_customer_date.

**Found :** 6 orders. All have both carrier and customer delivery timestamps. Five were purchased in October 2016, one in February 2018.

**Means :** These orders were dispatched and delivered, then marked cancelled. order_status and the delivery timestamps disagree. The October 2016 clustering suggests early-period data issues rather than a recurring problem.

**Action :** Decide in Silver which column wins — status or timestamps — and apply it consistently. Add a DQ test asserting no cancelled order has a delivery timestamp.

### F13 - order_status and the timestamp columns disagree on a small number of orders

**Table :** olist_orders_dataset.csv

**Checked :** Counted distinct order_id failing any of the four consistency conditions found in F9-F12.

**Found :** 29 distinct orders. By condition:
- 14 with status 'delivered' and no order_approved_at (F9)
- 2 with status 'delivered' and no order_delivered_carrier_date (F10)
- 8 with status 'delivered' and no order_delivered_customer_date (F11)
- 6 with status 'canceled' and a delivery timestamp (F12)

These sum to 30; one order fails two conditions (delivered, with both
carrier and customer timestamps missing), giving 29 distinct.

**Means :** Two sources of truth in one table disagree. The disagreement runs both ways — statuses claiming progress the timestamps don't record, and timestamps recording progress the status contradicts. Affected rows are under 0.1% of the table, but they sit in the columns every delivery and fulfilment metric is built from.

**Action :** Record in an ADR which column is authoritative when they conflict, and apply that choice consistently across all Silver logic — this is one decision, not four. Implement the four DQ tests from F9-F12. Route rows failing any of them to a separate review table rather than dropping them, so the conflict stays visible instead of disappearing.

### F14 - order_items has one row per item within an order

**Table :** olist_order_items_dataset.csv

**Checked :** Counted duplicate (order_id, order_item_id) pairs, checked each column on its own for uniqueness, and compared the min, max and count of order_item_id within each order.

**Found :** 112,650 rows, 0 duplicate pairs. Neither column is unique alone — order_id has 98,666 distinct values, order_item_id has 21. Within every order, order_item_id runs from 1 to the number of rows, with no gaps.

**Means :** The pair is a valid composite key, and a minimal one: drop either column and uniqueness fails. An order with three items has three rows — even when all three are the same product (see F16).

**Action :** Use (order_id, order_item_id) as the key in Silver. Any order-level number must be aggregated to order_id first.

### F15 - 775 orders have no line items

**Table :** olist_order_items_dataset.csv, olist_orders_dataset.csv

**Checked :** Found orders with no matching order_id in order_items using a left join from orders to items, filtered on null order_item_id. Broke the result down by order_status and compared each status against its total in orders.

**Found :** 775 orders (0.8%) have no items. By status, with the share of that status affected: created 5 (100%), unavailable 603 (99%), canceled 164 (26%), invoiced 2 (<1%), shipped 1 (<0.1%). No delivered, processing or approved order is missing items.

**Means :** 772 of these are orders that never progressed — created, unavailable, or cancelled before items were attached — so an empty order is expected. The 3 invoiced and shipped orders are inconsistent. All three were purchased on 5 October 2016 (a68ce168… shipped, handed to a carrier with nothing in it; 2ce96831… and e04f1da1… invoiced), the same early period as F12. No completed sale is affected.

**Action :** Always count orders from the orders table, never from a table joined to items. An inner join quietly drops these 775 orders, so any order count, cancellation rate or unavailability rate built on it would come out too low. Also add a check that every delivered, shipped or invoiced order has at least one item. Three orders fail it today — keep them visible as known issues rather than dropping them.

### F16 - One row is one unit: the same product repeats within an order

**Table :** olist_order_items_dataset.csv

**Checked :** Counted rows repeating an (order_id, product_id) pair, then counted distinct seller_id, shipping_limit_date, price and freight_value within each repeated pair.

**Found :** 7,088 order–product pairs appear on more than one row, adding 10,225 extra rows, across 6,968 orders (7.1% of orders with items). Within every pair, seller, shipping limit, price and freight are identical; only order_item_id changes.

**Means :** There is no quantity column. Three units of a product are three rows. A row is one unit, not one product line.

**Action :** Keep the unit-level grain in Silver. Derive quantity as the row count per (order_id, product_id). Count units with a row count and products with a distinct count of product_id — the two differ in 6,968 orders. Sum price over rows for an order's item value; never multiply price by the derived quantity — each row is already one unit.

### F17 - order_items links cleanly to orders and sellers

**Table :** olist_order_items_dataset.csv, olist_orders_dataset.csv, olist_sellers_dataset.csv

**Checked :** Compared the set of order_id in items against orders, and the set of seller_id in items against sellers, in both directions.

**Found :** 0 order_id in items are missing from orders; the other direction is F15's 775. 0 seller_id in items are missing from sellers, and 0 sellers have no items — both tables hold the same 3,095 sellers.

**Means :** Every item belongs to a known order and a known seller, and every seller in the sellers table has sold at least one item.

**Action :** Add DQ tests in Silver: every order_items.order_id must exist in orders, and every order_items.seller_id in sellers. Both are 0 today, so any failure later means new bad data — route those rows to quarantine rather than dropping them.

### F18 - An order can contain items from more than one seller

**Table :** olist_order_items_dataset.csv

**Checked :** Counted distinct seller_id per order_id.

**Found :** 1,278 of 98,666 orders with items (1.3%) have items from more than one seller: 1,219 have 2 sellers, 54 have 3, 3 have 4 and 2 have 5. The most in one order is 5.

**Means :** Seller belongs to the item row, not to the order — an order has no single seller. Counting orders per seller and adding them up gives more than the number of orders, because a multi-seller order counts once for each of its sellers.

**Action :** Keep seller_id on the item-level table in Silver; do not add a seller column to the order table. Any per-seller order figure must group by (order_id, seller_id).

## 3.Open Questions

### Q1 - Do all 32,951 product_ids in items exist in the products table?
To answer when products is profiled.

### Q2 - Why do 6 'unavailable' orders have items?
609 orders have status 'unavailable'; 603 of them have no items (F15).

### Q3 - Is zero freight real (free shipping) or missing?
383 rows in 339 orders (0.34% of rows) have freight_value 0.00, and they come from only 9 of 3,095 sellers. That concentration points to a seller-level practice such as free shipping rather than random gaps, but the items table alone can't confirm it.

### Q4 - Why do some shipping_limit_date values fall after the dataset ends?
The max is 2020-04-09; the latest date in the orders table is 2018-11-12. Rows not yet counted or explained — check in Silver before any on-time dispatch metric uses this column.