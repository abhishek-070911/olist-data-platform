# Data Profiling Report — Olist Brazilian E-Commerce

Source: [Kaggle, olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

Profiled by: Abhishek Patra

Notebook: [notebooks/01_profiling.ipynb](../notebooks/01_profiling.ipynb)

Last updated: 2026-10-09

## 1. Inventory

| File | Rows | Cols | Null Cols | Primary Key | PK Valid? |
|------|------|------|-----------|-------------|-----------|
| olist_sellers_dataset.csv | 3095 | 4 | 0 | seller_id | Yes — 3,095 distinct, no nulls |
| olist_customers_dataset.csv | 99441 | 5 | 0 | customer_id | Yes — 99,441 distinct, but order-scoped (see F5) |
| olist_orders_dataset.csv | 99441 | 8 | 3 | order_id | Yes — 99,441 distinct |
| olist_order_items_dataset.csv | 112650 | 7 | 0 | order_id,order_item_id | Yes — 112,650 distinct pairs, no nulls |
| olist_products_dataset.csv | 32951 | 9 | 8 | product_id | Yes — 32,951 distinct, no nulls |
| product_category_name_translation.csv | 71 | 2 | 0 | product_category_name | Yes — 71 distinct, no nulls |
| olist_order_payments_dataset.csv | 103886 | 5 | 0 | order_id,payment_sequential | Yes — 103,886 distinct pairs, no nulls |
| olist_order_reviews_dataset.csv | 99224 | 7 | 2 | review_id,order_id | Yes — 99,224 distinct pairs, no nulls |
| olist_geolocation_dataset.csv | 1000163 | 5 | 0 | none | No — no column is unique, and 261,831 rows are exact copies of another row (see F40)  |


### Column cardinality — sellers
| Column | Distinct | Note |
|--------|----------|------|
| seller_id | 3095 | = row count |
| seller_zip_code_prefix | 2246 | not unique per seller |
| seller_city | 611 | includes variant spellings (see F4) |
| seller_state | 23 | distinct codes present; includes DF (Federal District) | 

### Column cardinality — customers
| Column | Distinct | Note |
|--------|----------|------|
| customer_id | 99441 | = row count (see F5) |
| customer_unique_id | 96096 | 96,096 distinct across 99,441 rows — repeats (see F6) |
| customer_zip_code_prefix | 14994 | not unique per customer |
| customer_city | 4119 | not checked for variant spellings (see F4, F42) |
| customer_state | 27 | more codes than seller_state (23) |

### Column cardinality — orders
| Column | Distinct | Note |
|--------|----------|------|
| order_id | 99441 | Unique order id |
| customer_id | 99441 | Same number of rows as of customers table |
| order_status | 8 | approved,canceled,created,delivered,invoiced,processing,shipped,unavailable |
| order_purchase_timestamp | 98875 | 566 fewer distinct times than orders, so some orders share a purchase time |
| order_approved_at | 90733 | 160 Null rows |
| order_delivered_carrier_date | 81018 | 1783 Null rows |
| order_delivered_customer_date | 95664 | 2965 Null rows |
| order_estimated_delivery_date| 459 | date only; no time part, unlike the other 4 date columns |

### Column cardinality — items
| Column | Distinct | Note |
|--------|----------|------|
| order_id | 98666 | repeats — up to 21 rows per order; 775 orders have no items (see F15) |
| order_item_id | 21 | position within an order; numbered 1, 2, 3… up to the order's row count, no gaps; max 21 (see F14) |
| product_id | 32951 | same 32,951 products as the products table, both directions (see F19) |
| seller_id | 3095 | same 3,095 sellers as the sellers table, both directions (see F17) |
| shipping_limit_date | 93318 | max 2020-04-09, later than every date in the orders table — not investigated yet (see Q4) |
| price | 5968 | min 0.85, max 6,735.00 — no zero or negative prices |
| freight_value | 6999 | min 0.00 — 383 rows in 339 orders have zero freight (see Q3) |

### Column cardinality — products
| Column | Distinct | Note |
|--------|----------|------|
| product_id | 32951 | = row count; same 32,951 products as the items table (see F19) | 
| product_category_name | 73 | 73 categories; 610 products have none (see F20); 2 have no English name (see F24) |
| product_name_lenght | 66 | 5 to 76 characters; 610 empty (see F20, F23) |
| product_description_lenght | 2960 | 4 to 3,992 characters; 610 empty (see F20, F23) |
| product_photos_qty | 19 | 1 to 20 photos; 610 empty (see F20, F23) |
| product_weight_g | 2204 | 0 to 40,425 g; 2 empty (see F21); 4 are 0 g (see F22) |
| product_length_cm | 99 | 7 to 105 cm; 2 empty (see F21) |
| product_height_cm | 102 | 2 to 105 cm; 2 empty (see F21) |
| product_width_cm | 95 | 6 to 118 cm; 2 empty (see F21) |

### Column cardinality — name translation
| Column | Distinct | Note |
|--------|----------|------|
| product_category_name | 71 | = row count; one row per category (the key); covers 71 of the 73 product categories (see F24) |
| product_category_name_english | 71 | = row count; every category has its own English name |

### Column cardinality — payments
| Column | Distinct | Note |
|--------|----------|------|
| order_id | 99440 | repeats — up to 29 rows per order; 1 order in orders has no payment (see F25) |
| payment_sequential | 29 | 1 to 29; in 80 orders the numbering starts at 2 instead of 1 (see F27) |
| payment_type | 5 | credit_card 76,795; boleto 19,784; voucher 5,775; debit_card 1,529; not_defined 3 |
| payment_installments | 24 | 0 to 24, every value except 19; 2 rows have 0, both in delivered orders (see F29) |
| payment_value | 29077 | 0.00 to 13,664.08; 9 rows are 0.00 (see F26) |

### Column cardinality — reviews
| Column | Distinct | Note |
|--------|----------|------|
| review_id | 98410 | repeats — 789 review_ids appear on 2 or 3 orders; copies are identical apart from order_id (see F31) |
| order_id | 98673 | repeats — 547 orders have 2 or 3 reviews (see F32); 768 orders have no review (see F33) |
| review_score | 5 | 1 to 5; 57.8% are 5 and 11.5% are 1 (see F39) |
| review_comment_title | 4527 | empty in 87,656 rows (88%) (see F34) |
| review_comment_message | 36159 | empty in 58,247 rows (59%) (see F34) |
| review_creation_date | 636 | date only, 2 Oct 2016 to 31 Aug 2018; 85 rows have a time part (see F35) |
| review_answer_timestamp | 98248 | 7 Oct 2016 to 29 Oct 2018; never before the creation date (see F36) |

### Column cardinality — geolocation
| Column | Distinct | Note |
|--------|----------|------|
| geolocation_zip_code_prefix | 19015 | read as text so leading zeros stay (see F3); 1 to 1,146 rows per prefix, median 29 (see F40); 157 customer and 7 seller prefixes are missing (see F43) |
| geolocation_lat | 717360 | -36.61 to 45.07; both ends are outside Brazil (see Q7) |
| geolocation_lng | 717613 | -101.47 to 121.11; both ends are outside Brazil (see Q7) |
| geolocation_city | 8011 | 5,968 once accents are removed; 8,556 prefixes have more than one city name (see F42) |
| geolocation_state | 27 | same number of codes as customer_state; 8 prefixes have two states (see F41) |

### Relationships between tables

| Child → Parent | Key | Child rows with no parent | Parent rows with no child | See |
|----------------|-----|---------------------------|---------------------------|-----|
| orders → customers | customer_id | 0 | 0 | F8 |
| order_items → orders | order_id | 0 | 775 orders have no items | F15, F17 |
| order_items → sellers | seller_id | 0 | 0 | F17 |
| order_items → products | product_id | 0 | 0 | F19 |
| products → name translation | product_category_name | 2 categories (13 products); 610 products have no category | 0 | F20, F24 |
| payments → orders | order_id | 0 | 1 order has no payment | F25 |
| reviews → orders | order_id | 0 | 768 orders have no review | F33 |
| customers → geolocation | zip prefix | 157 prefixes (278 orders) | 4,099 prefixes used by no customer or seller | F40, F43 |
| sellers → geolocation | zip prefix | 7 prefixes (7 sellers) | same 4,099 | F40, F43 |

## 2. Findings

### F1 - seller_zip_code_prefix is not unique per seller

**Table :** olist_sellers_dataset.csv.

**Checked :** Distinct 'seller_zip_code_prefix' vs row count.

**Found :** 2,246 distinct values across 3,095 sellers. The most common prefix covers 49 sellers.

**Means :** Multiple sellers share same zip code, so it identifies 'location' and not a seller.

**Action :** Treat it as descriptive attribute, not as a key.

### F2 - 60% of sellers are in SP, and 17 of 23 states have fewer than 50 sellers each

**Table :** olist_sellers_dataset.csv.

**Checked :** Counted sellers per state (value_counts on seller_state).

**Found :** SP holds 1,849 of 3,095 sellers (60%); top 3 (SP, PR, MG) hold 2,442 (79%). 17 of the 23 state codes have fewer than 50 sellers each, and 5 of them (AC, AM, MA, PA, PI) have a single seller.

**Means :** Sellers are spread very unevenly: SP alone has more sellers than all other states together (1,849 vs 1,246). In most states, a per-state seller figure rests on a handful of sellers, and in 5 states on just one, so it can swing a lot and describes that seller more than the state.

**Action :** In Gold, show the number of sellers next to any per-state seller metric (such as average review score or delivery time), so figures from states with few sellers are not read as reliable.

### F3 - Zip prefix is a 5-character code with leading zeros, so it must be read as text

**Table :** olist_sellers_dataset.csv, olist_customers_dataset.csv.

**Checked :** Read the zip prefix columns as text (dtype=str) and counted the values by length.

**Found :** All 3,095 seller prefixes and all 99,441 customer prefixes have 5 characters. Values that start with 0 keep it, such as 04195 (sellers) and 01151 (customers).

**Means :** The zip prefix is a code, not a number: every value has exactly 5 characters, and the leading zero is part of it. Read without dtype=str, pandas would treat the column as numbers (int64), and a number cannot keep a leading zero, so 04195 would become 4195 and stop matching any table or zip list that keeps it as text. This is about how we read the file, not a problem in the source data.

**Action :** Read zip prefix columns with dtype=str at ingestion, in sellers, customers and geolocation. Add a DQ test asserting length == 5; today every seller and customer prefix passes.

### F4 - seller_city contains variant spellings of the same city

**Table :** olist_sellers_dataset.csv.

**Checked :** Filtered seller_city for 'do rio preto'.

**Found :** 'sao jose do rio preto' (33) and 's jose do rio preto' (1) — the same city under two values.

**Means :** Any group-by on raw city splits this city in two. Other abbreviations likely exist across the 611 values. Geolocation has the same problem on a much larger scale (see F42).

**Action :** Do not use raw city as a grouping key. Standardise in Silver against a municipality reference list. Extent of the problem across all 611 values not yet measured.


### F5 - customer_id is unique per row but does not identify a customer

**Table :** olist_customers_dataset.csv.

**Checked :** Compared distinct customer_id and customer_unique_id against the row count.

**Found :** 99,441 distinct customer_id across 99,441 rows; 96,096 distinct customer_unique_id.

**Means :** customer_id is created for each order, so it identifies an order, not a person. The Kaggle data description says so, and F8 confirms it from the data.

**Action :** Do not use customer_id to count or identify customers.

### F6 - 2,997 customers (3.1%) placed more than one order

**Table :** olist_customers_dataset.csv.

**Checked :** Counted customer_unique_id values appearing on more than one row.

**Found :** 2,997 of 96,096 customer_unique_id values (3.1%) appear on more than one row.

**Means :** One row is one order (F8), so these 2,997 customers placed more than one order. customer_id cannot show this, because a new one is created for every order.

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

### F19 - order_items links cleanly to products, in both directions

**Table :** olist_order_items_dataset.csv, olist_products_dataset.csv

**Checked :** Compared the set of product_id in items against products, in both directions.

**Found :** 32,951 distinct product_id in items and 32,951 in products. 0 are in items but not in products, and 0 are in products but not in items.

**Means :** Every item links to a known product, and every product in the products table appears in at least one order item.

**Action :** Add a DQ test in Silver: every order_items.product_id must exist in products. It is 0 today, so any failure later means new bad data — route those rows to quarantine rather than dropping them.

### F20 - 610 products are missing their whole description, not just single fields

**Table :** olist_products_dataset.csv

**Checked :** Filtered products where product_category_name is null, then counted nulls in every other column for just those rows.

**Found :** 610 of 32,951 products (1.9%) have a null product_category_name. All 610 also have null product_name_lenght, product_description_lenght and product_photos_qty, so these four columns are empty together, on the same products. Inside these 610, weight, length, height and width have 1 null each (2 each in the whole table). These 610 products appear in 1,603 item rows (1.4% of all item rows) across 1,451 orders (1.5% of orders with items).

**Means :** For these 610 products the whole description is missing. It is one block of missing information, not four separate problems. The products are real: every one of them appears in at least one order item (F19). A category report will either leave out their sales or show them under a blank category, and any average of photos or text length will quietly skip them.

**Action :** Keep all 610 products and never drop them, because they appear in real orders. In Silver, label their category as 'unknown' so category reports show them as their own line instead of losing them. Leave name length, description length and photos empty; do not fill them with 0 or an average, because that would invent values. Add a DQ check that counts products with no category, with 610 as today's baseline, so the pipeline flags it if the number grows.

### F21 - 2 products have no weight or size

**Table :** olist_products_dataset.csv

**Checked :** Picked the products with an empty weight, then checked which of their other columns were also empty.

**Found :** 2 of 32,951 products have no weight. Both also have no length, height or width. One of them (09ff539a…, category 'bebes') still has its category, name length, description length and photos. The other (5eb56465…) has nothing filled in except its product_id. It is also one of the 610 products in F20.

**Means :** For these 2 products, weight and all three size measurements are missing together. The second product has no information at all apart from its id. Both are real products that show up in orders (F19), but we cannot use them in anything that needs weight or size.

**Action :** Keep both products and do not delete them. Leave their weight and size empty; do not fill in 0 or an average, because that would be a made-up number. When calculating anything with weight or size, skip these 2 products instead of counting them as 0. Add a DQ check that counts products with no weight; today the count is 2, so the check should warn if it goes up.

### F22 - 4 products have a weight of 0 g

**Table :** olist_products_dataset.csv

**Checked :** Filtered products where product_weight_g is 0 and looked at all their columns.

**Found :** 4 of 32,951 products have product_weight_g = 0. All 4 are in the category cama_mesa_banho, all measure 30 × 25 × 30 cm (length × height × width), each has 1 photo, and their description lengths are 528 to 529 characters.

**Means :** A real product cannot weigh 0 g, so these 0s are not real weights; they behave like missing values written as a number. That makes them worse than an empty cell, because 0 looks real: it pulls down any average weight and breaks any calculation that divides by weight. The 4 look like versions of the same listing — same category, same size and almost the same description length.

**Action :** In Silver, treat a weight of 0 as missing: store it as empty, not 0, so these 4 products are handled the same way as the 2 in F21. Add a DQ test that product_weight_g must be above 0 whenever it is filled in; 4 products fail it today.

### F23 - Two column names are spelled wrong, and three counts are stored as decimals

**Table :** olist_products_dataset.csv

**Checked :** Looked at the column names and data types with info(), and at the values in the first few rows.

**Found :** Two column names say "lenght" instead of "length": product_name_lenght and product_description_lenght. These two columns and product_photos_qty are stored as decimals (float64), so whole numbers show up as 40.0, 287.0 and 1.0. All three have 610 empty values (F20).

**Means :** The spelling mistake is in the source file itself, not in our code, and it is easy to type "length" by habit and get an error. The decimals happen because pandas cannot keep empty values in a whole-number column, so when even one value is empty it turns the whole column into decimals. These columns count things (characters and photos), so they can only be whole numbers; the .0 comes from the way the file was read, not from the data. It is the same kind of problem as F3: the data is fine, and the way we read it needs fixing.

**Action :** In Bronze, keep the column names exactly as they are in the source file, so we can always trace back to the original. In Silver, rename them to product_name_length and product_description_length. When reading the three count columns, use a whole-number type that allows empty values (Int64 in pandas, INTEGER in the database), so the numbers stay whole and the 610 empty values stay empty.

### F24 - 2 product categories have no English name

**Table :** olist_products_dataset.csv, product_category_name_translation.csv

**Checked :** Compared the categories in products (empty ones left out) with the categories in the translation table, in both directions, then counted the products and order items in the categories that did not match.

**Found :** 71 of the 73 categories in products have an English name. 2 do not: portateis_cozinha_e_preparadores_de_alimentos (10 products) and pc_gamer (3 products). These 13 products appear in 24 item rows across 22 orders. In the other direction, every category in the translation table is used by at least one product.

**Means :** The translation table is almost complete, but not quite. If products are joined to translation with an inner join, these 13 products and their sales silently disappear from any report that uses English names. With a left join they stay, but their English name is empty.

**Action :** In Silver, join products to translation with a left join so no product is lost. Where there is no English name, use the Portuguese name instead. Products with no category at all are labelled 'unknown' (F20). Add a DQ test that every non-empty category in products has a translation; 2 fail today.

### F25 - 1 delivered order has no payment record

**Table :** olist_orders_dataset.csv, olist_order_payments_dataset.csv, olist_order_items_dataset.csv

**Checked :** Compared the order_ids in orders and payments, both ways, looked at the one order that did not match, then looked it up in the items table.

**Found :** 1 order 'bfbd0f9bdef84302105ad712db648a6c' is in orders but has no payment. It was delivered: bought on 15 Sep 2016, approved in the same second, given to the carrier on 7 Nov 2016 and delivered on 9 Nov 2016. Every order in payments is also in orders. It has 3 rows in the items table: 3 units of the same product from one seller (see F16), each 44.99 plus 2.83 freight, 143.46 in total.

**Means :** Every order has a payment except this one. A delivered order should have been paid for, so its payment record was either lost or never saved. It comes from September 2016, the same early period as F12 and F15. Any revenue number worked out from payments will miss this order.

**Action :** Keep this order and do not delete it. Add a DQ check that every delivered order has at least one payment; today 1 order fails. When calculating revenue, write down which table it comes from (payments or items), because the two may give different answers for orders like this one.

### F26 - 9 payments are worth 0: 3 "not_defined" payments on cancelled orders and 6 vouchers worth nothing

**Table :** olist_order_payments_dataset.csv, olist_orders_dataset.csv

**Checked :** Picked the payment rows with a value of 0, then looked up the status of their orders in the orders table.

**Found :** 9 payment rows, in 8 orders, have a value of 0.00. 3 of them have the payment type not_defined. All 3 belong to orders that were cancelled before they were ever approved (00b1cb03…, 4637ca19…, c8c52818…, bought in August and September 2018). The other 6 are vouchers in 5 orders: 4 delivered and 1 shipped. Their sequence numbers are 3, 4, 13 and 14, so each of these orders also has other payments; the zero voucher is just one of several payments in the order.

**Means :** These are two different cases. The not_defined rows belong to orders that were cancelled before any payment was approved, so the 0 is correct: no money was paid. The zero vouchers sit inside orders that were paid in other ways. They do not change the order total, so sums stay right, but they make the number of payments look bigger and pull down the average payment value.

**Action :** Keep all 9 rows. Keep not_defined as its own payment type; do not mix it into another type. When counting payments or working out the average payment value, leave out the rows with a value of 0. Add DQ checks that payment_type is always one of the 5 known types and that payment_value is never below 0. Today there are 9 zero rows and 3 not_defined rows, so the checks should warn if these numbers go up.

### F27 - 80 orders have payment numbers that start at 2 instead of 1

**Table :** olist_order_payments_dataset.csv, olist_order_items_dataset.csv

**Checked :** Checked whether payment_sequential runs 1, 2, 3 and so on without gaps in every order, then grouped the flagged orders by their (min, max, count) pattern. For the flagged orders, compared each order's payment total with its item total (see F30) and counted their payment types.

**Found :** 80 orders have payment numbers that start at 2 instead of 1: 78 have a single payment numbered 2, and 2 have payments numbered 2 and 3. No order skips a number in the middle. In 79 of them the payments add up to the items: 77 exactly and 2 within 1 cent. The other one has no items at all (see F28). Their 82 payment rows are mostly debit card (52), then credit card (29) and boleto (1). The 2 zero-installment payments in F29 are in these orders.

**Means :** No money is missing: in these orders the payments that are there already add up to the items. So either there never was a payment number 1, or it was worth nothing; either way the totals are right. Debit card makes up 52 of these 82 payment rows but only 1.5% of all payment rows, so the numbering seems tied to how some debit card payments were recorded. Anything that looks for payment number 1 to find an order's first or main payment will find nothing for these 80 orders.

**Action :** Keep the rows and their numbers as they are; do not renumber them. To find an order's first payment, use the lowest payment_sequential in the order, not payment_sequential = 1. Add a DQ check that counts orders whose numbering does not start at 1; today it is 80, so it should warn if this goes up.


### F28 - 775 orders with no items still have payments worth 162,591.95

**Table :** olist_order_payments_dataset.csv, olist_order_items_dataset.csv, olist_orders_dataset.csv

**Checked :** Compared the order_ids in items and payments, both ways. Checked that the orders with payments but no items are the same orders as in F15, then added up their payments by order status.

**Found :** 1 order has items but no payment (bfbd0f9b…, see F25). 775 orders have payments but no items. They are exactly the 775 orders with no items in F15, so every order without items still has a payment record. Together they have 830 payment rows worth 162,591.95: unavailable 603 orders (124,339.02), canceled 164 (37,337.87), created 5 (688.10), invoiced 2 (149.23) and shipped 1 (77.73).

**Means :** The payments table holds money for orders that have nothing in them. Almost all of it is on unavailable and cancelled orders, and nothing in the payments table says whether it was refunded. So a payment row does not prove a sale. A revenue total that adds up all of payment_value would include these 162,591.95 (about 1% of all payment value); a total built from items would not. This is the other side of F25, where an order has items but no payment.

**Action :** Keep all 830 payment rows; do not delete them. In Silver, give every order two flags: has_items and has_payment. Count revenue only for orders where has_items is true, and show the money on orders without items as its own number, split by status. Add a DQ check that counts orders with payments but no items; today it is 775 orders and 162,591.95, so it should warn if either goes up.

### F29 - 2 credit card payments have 0 installments

**Table :** olist_order_payments_dataset.csv, olist_orders_dataset.csv

**Checked :** Listed the distinct values of payment_installments, then looked at the rows with 0 and the status of their orders.

**Found :** payment_installments has every value from 0 to 24 except 19. 2 rows have 0 installments, one in each of 2 delivered orders (744bade1…, 1a571083…, bought in April and May 2018). Both are credit card payments, numbered 2 in their order, worth 58.69 and 129.94. Both orders are also among the 80 orders in F27 whose payment numbers start at 2.

**Means :** Every other payment row has at least 1 installment, and a payment can't be split into 0 parts, so these 2 zeros are recording errors. The payments themselves look real: they have a value and both orders were delivered. Anything that divides by the number of installments, such as the amount of each installment, would divide by 0 for these 2 rows.

**Action :** Keep both rows. Bronze keeps the 0 as it is. In Silver, store it as null (unknown) and flag the row, so calculations skip it instead of breaking. Add a DQ check that payment_installments is at least 1; today 2 rows fail, so it should warn if this goes up.

### F30 - Payments match price + freight for 99.4% of orders

**Table :** olist_order_payments_dataset.csv, olist_order_items_dataset.csv

**Checked :** Added up each order's payment_value, and each order's price + freight_value, joined the two totals on order_id (keeping orders found in only one table), and compared them to the cent.

**Found :** 98,665 orders are in both tables; 776 are in only one (775 with payments but no items, see F28; 1 with items but no payment, see F25). Of the 98,665: 98,089 (99.4%) match exactly, 273 are off by 1 cent, 264 have payments larger than their items (by 0.02 to 182.81, median about 6.5), and 39 have payments smaller than their items (by 0.02 to 51.62, median 0.04).

**Means :** For almost every order, what the customer paid equals the price plus freight, so the two tables agree. A 1-cent difference is rounding, not missing money. The 303 orders that differ by more than 1 cent are real gaps that this check can't explain yet (see Q5). Because the two tables agree this closely, items can be the source for sales revenue, with payments kept as the amount actually paid.

**Action :** Build sales revenue from items (price + freight_value) and write this down as the project's revenue definition; this settles the choice F25 asked for. Keep each order's payment total next to it as the amount paid, and store the difference. Treat a difference of 1 cent or less as a match. Add a DQ check on the difference: today 303 orders differ by more than 1 cent (264 more, 39 less), so it should warn if this goes up.

### F31 - The same review is copied onto 2 or 3 orders

**Table :** olist_order_reviews_dataset.csv

**Checked :** Counted how often each review_id appears. For the review_ids that repeat, counted how many different values every other column has across the copies.

**Found :** 99,224 rows but only 98,410 distinct review_ids. 789 review_ids appear more than once: 764 twice and 25 three times, which accounts for all 814 extra rows. In all 789, the copies have the same score, title, message, creation date and answer time; only order_id differs. No (review_id, order_id) pair repeats.

**Means :** One review was attached to several orders, so review_id alone is not a key: one row is one review for one order. Counting rows gives 814 more reviews than were actually written; counting distinct review_ids gives the real number.

**Action :** Use (review_id, order_id) as the key. Count distinct review_id when counting reviews or averaging scores per review; use the pair when attaching a score to an order. Add DQ checks that the pair stays unique and that copies of a review_id never disagree; today 789 review_ids repeat and all copies match.

### F32 - 547 orders have more than one review, and 202 of them disagree on the score

**Table :** olist_order_reviews_dataset.csv

**Checked :** Counted reviews per order. For orders with more than one review, counted how many different values each column has, and for orders whose scores differ, measured the gap between the lowest and highest score.

**Found :** 98,126 orders have 1 review, 543 have 2 and 4 have 3 (547 orders, 1,098 rows). Within those 547 orders, the answer time differs in all 547, the creation date in 392, the message in 230, the score in 202 and the title in 13. In the 202 orders with different scores, the gap is 1 point in 90, 2 in 47, 3 in 32 and 4 in 33.

**Means :** These are separate reviews, not copies (unlike F31). An order has no single score: 202 orders (37% of multi-review orders) have conflicting scores, and 65 of them are 3 or 4 points apart, such as 1 and 5. Because the answer times always differ, the latest review can be picked.

**Action :** Keep every review row in Silver. For order-level metrics in Gold, use one review per order: the one with the latest review_answer_timestamp, and write this rule in the model's documentation. Add a DQ check that counts orders with more than one review; today it is 547, so it should warn if this goes up.

### F33 - Every review links to an order, but 768 orders have no review

**Table :** olist_order_reviews_dataset.csv, olist_orders_dataset.csv

**Checked :** Compared the order_ids in reviews and orders, both ways, then looked at the status of the orders that have no review.

**Found :** Every order_id in reviews exists in orders. 768 of 99,441 orders (0.8%) have no review: delivered 646, shipped 75, canceled 20, unavailable 14, processing 6, invoiced 5, created 2.

**Means :** Reviews link cleanly to orders. Most orders without a review were delivered, so a missing review means the customer didn't answer, not a broken link. These orders have no score at all; they are not unhappy customers.

**Action :** Join from orders to reviews with a left join so the 768 orders stay visible with an empty score; never fill it with 0. Calculate review rates and average scores only over orders that have a review. Add a DQ check that every review's order_id exists in orders; today 0 fail.

### F34 - Most reviews have a score but no written comment

**Table :** olist_order_reviews_dataset.csv

**Checked :** Counted reviews by which comment columns are filled: title only, message only, both, or neither.

**Found :** review_comment_title is empty in 87,656 rows (88%) and review_comment_message in 58,247 (59%). 56,518 reviews (57%) have neither, 31,138 have only a message, 1,729 only a title and 9,839 both. Every review has a score.

**Means :** Comments are optional: a review is a score first, and text is extra. An empty title or message means nothing was written, not that data was lost. Only 43% of reviews (42,706) have any text, so any analysis of comments covers less than half of the reviews.

**Action :** Keep empty comments as nulls; do not fill them with blank text. In Silver, add a flag has_comment (true if the title or message is filled). Do not drop reviews without text from score metrics.

### F35 - review_creation_date is a date, except for 85 rows

**Table :** olist_order_reviews_dataset.csv

**Checked :** Compared every review_creation_date with its normalised (midnight) version and counted the rows that change.

**Found :** 99,139 creation dates are at midnight and 85 are not. The column has 636 distinct days, from 2 Oct 2016 to 31 Aug 2018.

**Means :** The column holds a date, not a time; the 85 rows are the exception, and their cause is not yet known (see Q6). Any comparison with a full timestamp, such as the purchase or delivery time, has to compare dates only, or same-day events look out of order.

**Action :** In Silver, store review_creation_date as a date and compare it with other timestamps on the date only. Add a DQ check that counts creation dates with a time part; today it is 85.

### F36 - Reviews are answered after they are created, mostly within 3 days

**Table :** olist_order_reviews_dataset.csv

**Checked :** Subtracted review_creation_date from review_answer_timestamp and looked at the gap in days.

**Found :** No answer comes before its review was created; the smallest gap is 0 days. Half of the reviews are answered by the next day and three-quarters within 3 days. 658 reviews (0.7%) were answered more than 30 days later, the longest after 518 days.

**Means :** The two columns are in the right order in every row. Most customers answer quickly, but a small group answers weeks or months later, so a review's creation date and answer date can fall in very different months.

**Action :** Add a DQ check that review_answer_timestamp is never before review_creation_date; today 0 fail. When reporting scores by month, choose either the creation date or the answer date and write the choice down.

### F37 - 64 reviews were created before their order was bought

**Table :** olist_order_reviews_dataset.csv, olist_orders_dataset.csv

**Checked :** Compared each review's creation date with its order's purchase date (dates only), then looked at the status of those orders and how many days early the reviews were.

**Found :** 64 reviews were created before their order was bought: 57 on canceled orders, 6 on delivered and 1 on shipped. They are 1 to 111 days early; the median is 15.5 days.

**Means :** A review can't be written before the purchase, so one of the two dates is wrong for these orders, and the data does not show which one. Almost all are cancelled orders, so it mostly touches orders that never completed.

**Action :** Keep these rows and flag them in Silver (review_before_purchase). Leave them out of any "time from purchase to review" metric. Add a DQ check that review_creation_date is not before the purchase date; today 64 fail, so it should warn if this goes up.

### F38 - 8% of reviews were written before the order arrived

**Table :** olist_order_reviews_dataset.csv, olist_orders_dataset.csv

**Checked :** Compared each review's creation date with its order's delivery date (dates only), counted reviews whose order has no delivery date, and for both groups measured the days between the estimated delivery date and the review's creation date.

**Found :** 5,127 reviews were created before the order was delivered (delivered 5,126, canceled 1). Another 2,865 belong to orders with no delivery date: shipped 1,043, canceled 603, unavailable 597, invoiced 313, processing 296, delivered 8 (see F11), created 3, approved 2. Of these 7,992 early reviews, 5,248 (66%) were created exactly 2 days after the estimated delivery date; the next most common gaps are 3 days (962), 4 (352) and 1 (314).

**Means :** The review survey is not sent only after delivery. When an order is late or never arrives, the review appears about 2 days after the estimated delivery date. So 8% of reviews (7,992 of 99,224) were written before the customer had the goods; those scores rate the delay, not the product.

**Action :** In Silver, add a flag reviewed_before_delivery (true for these 7,992). Report scores from reviews written after delivery separately from those written before it. Compare review dates with delivery dates on the date only, and treat an empty delivery date as "not delivered".

### F39 - Scores lean heavily towards 5, and undelivered orders score far lower

**Table :** olist_order_reviews_dataset.csv, olist_orders_dataset.csv

**Checked :** Counted reviews per score, then the number of reviews and the average score for each order status.

**Found :** Score 5: 57.8%, 4: 19.3%, 3: 8.2%, 2: 3.2%, 1: 11.5%. Every score is between 1 and 5. Delivered orders (96,361 reviews) average 4.16. All other statuses together have 2,863 reviews, averaging between 1.28 and 2.50: shipped 2.01 (1,043), canceled 1.81 (609), unavailable 1.53 (597), invoiced 1.66 (313), processing 1.28 (296), created 2.33 (3), approved 2.50 (2). These counts are review rows, so the 814 copied rows from F31 are included.

**Means :** Scores are not spread evenly: most customers give 5, and unhappy customers give 1 far more often than 2. Reviews on orders that never arrived rate the failed delivery, not the product, and they pull any overall average down. An average on its own hides this shape.

**Action :** Calculate product and seller scores from delivered orders only, and report scores for undelivered orders as their own number. Show the share of 4–5 and 1–2 scores next to the average. Add a DQ check that review_score is always between 1 and 5; today all rows pass.

### F40 - 26% of geolocation rows are exact copies, and zip prefixes still repeat without them

**Table :** olist_geolocation_dataset.csv

**Checked :** Counted rows that exactly repeat an earlier row (all 5 columns the same). Then counted rows per zip prefix, first on all rows and then with the exact copies removed.

**Found :** 261,831 of 1,000,163 rows (26%) are exact copies of another row; 738,332 rows are left without them. Rows per zip prefix, all rows: median 29, average 53, largest 1,146. Without the copies: median 23, average 39, largest 779. All 19,015 zip prefixes remain either way.

**Means :** The copies add no information: removing them loses nothing, because each one repeats a row that stays. But even without them, one zip prefix has up to 779 rows, so this table is not one row per zip prefix. Joining customers or sellers to it on zip prefix would multiply their rows.

**Action :** In Silver, drop exact copies, then reduce the table to one row per zip prefix (for example the median latitude and longitude) before joining it to customers or sellers. Add DQ checks: count exact copies (today 261,831) and check there is one row per zip prefix after the reduction.

### F41 - 8 zip prefixes appear in two states, each because of one stray row

**Table :** olist_geolocation_dataset.csv

**Checked :** Counted distinct states per zip prefix, then counted rows per state for the prefixes with more than one.

**Found :** 8 of 19,015 zip prefixes have two states. In each one, one state has a single row and the other has 12 to 179 rows: 02116 (SP 12, RN 1), 04011 (SP 178, AC 1), 21550 (RJ 170, AC 1), 23056 (RJ 60, AC 1), 72915 (GO 40, DF 1), 78557 (MT 96, RO 1), 79750 (MS 179, RS 1), 80630 (PR 122, SC 1).

**Means :** A zip prefix belongs to one state, so in each of these 8 the single row is almost certainly the mistake. Brazil gives out zip codes in ranges per state (external fact, Correios), and in all 8 the state with more rows is the one that owns the range. If the table is reduced to one row per prefix by taking any row, these prefixes can end up in the wrong state.

**Action :** When reducing the table to one row per zip prefix (F40), take the state with the most rows for that prefix. Add a DQ check that counts zip prefixes with more than one state; today it is 8, so it should warn if this goes up.

### F42 - 45% of zip prefixes have more than one city name, mostly the same city with and without accents

**Table :** olist_geolocation_dataset.csv

**Checked :** Counted distinct city names per zip prefix and looked at a random sample of 5. Then removed accents from the city names in a separate column (the original is unchanged), counted again, and looked at a sample of the prefixes that still had more than one name.

**Found :** 8,556 of 19,015 zip prefixes (45%) have more than one city name: 8,265 have 2, 255 have 3, 27 have 4 and 9 have 5. Each of the 5 sampled prefixes had the same city written with and without accents, such as "sao paulo" (76 rows) and "são paulo" (14). With accents removed, distinct city names drop from 8,011 to 5,968, and prefixes with more than one name drop from 8,556 to 550, so 8,006 (94%) differed only by accents. A sample of 5 of the 550 showed three other kinds: punctuation ("santa barbara d'oeste", "d oeste", "doeste"), one stray row naming a nearby city (nilopolis 199 rows, rio de janeiro 1), and a district written instead of its city (pipa and tibau do sul; ceilandia and brasilia).

**Means :** Most of these are one city written two ways, the same kind of problem as F4 in sellers. Grouping on the raw name splits one city into several, so São Paulo is counted under two names. Removing accents fixes 94% but not the rest, and picking the most common name per prefix is not always right either: in 59179 it would give pipa, a district of tibau do sul (external fact).

**Action :** In Silver, clean city names with one rule shared by geolocation, customers and sellers: lowercase, remove accents, and make apostrophes and spaces consistent. When reducing geolocation to one row per zip prefix (F40), keep the most common cleaned name, and correct district names with the municipality list from F4. Join on zip prefix, never on city name. Add a DQ check that counts prefixes with more than one cleaned city name; today it is 550.

### F43 - 278 orders and 7 sellers have a zip prefix with no location

**Table :** olist_geolocation_dataset.csv, olist_customers_dataset.csv, olist_sellers_dataset.csv

**Checked :** Compared the zip prefixes in customers and sellers with those in geolocation, both ways, then counted the customer and seller rows whose prefix has no match.

**Found :** 157 of 14,994 customer zip prefixes and 7 of 2,246 seller zip prefixes are not in geolocation. They cover 278 customer rows, which are 278 orders because each customer_id is one order (F8), and 7 sellers. In the other direction, 4,099 of the 19,015 geolocation prefixes are not used by any customer or seller.

**Means :** 99.7% of orders and 99.8% of sellers can be given coordinates. The rest still have their own city and state, because the customers and sellers tables have no nulls, so only latitude and longitude are missing. The 4,099 unused prefixes do no harm: geolocation simply covers more zip prefixes than customers and sellers use.

**Action :** Join customers and sellers to the reduced geolocation table (F40) with a left join, so the 278 orders and 7 sellers stay with empty coordinates; never use an inner join here. In Silver, add a has_location flag. Add DQ checks that count customer and seller zip prefixes with no location; today 157 (278 orders) and 7, so they should warn if these go up.

## 3. Open Questions

### Q1 - Do all 32,951 product_ids in items exist in the products table?
**Answered (F19):** yes — all 32,951 match, in both directions.

### Q2 - Why do 6 'unavailable' orders have items?
609 orders have status 'unavailable'; 603 of them have no items (F15). Next: look at the timestamps, items and payments of those 6 orders.

### Q3 - Is zero freight real (free shipping) or missing?
383 rows in 339 orders (0.34% of rows) have freight_value 0.00, and they come from only 9 of 3,095 sellers. That concentration points to a seller-level practice such as free shipping rather than random gaps, but the items table alone can't confirm it. Next: for these 9 sellers, compare their zero-freight rows with their other rows (dates, products, customer states).

### Q4 - Why do some shipping_limit_date values fall after the dataset ends?
The max is 2020-04-09; the latest date in the orders table is 2018-11-12. Rows not yet counted or explained — check in Silver before any on-time dispatch metric uses this column.

### Q5 - Why do 303 orders have payments that differ from their items by more than 1 cent?
264 orders have payments larger than price + freight (by up to 182.81) and 39 have smaller (by up to 51.62) (see F30). The totals alone don't say why. Next: look at the payment types and installments of these orders.

### Q6 - Why do 85 review creation dates have a time part?
All other 99,139 creation dates are at midnight (see F35). Next: look at the times and dates of those 85 rows, and compare the dates with when Brazil's clocks moved forward for daylight saving in 2016 and 2017.

### Q7 - How many geolocation points fall outside Brazil?
describe() shows latitudes up to 45.07 and longitudes from -101.47 to 121.11, while Brazil lies roughly between latitude +5 and -34 and longitude -74 and -35 (external fact). These rows are not counted yet. Check before any map or distance metric uses the coordinates; the median point per zip prefix (F40 Action) limits their effect.