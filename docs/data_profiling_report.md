# Data Profiling Report — Olist Brazilian E-Commerce

Source: [Kaggle, olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

Profiled by: Abhishek Patra

Last updated: 2026-09-14

## 1.Inventory

| File | Rows | Cols | Null Cols | Primary Key | PK Valid? |
|------|------|------|-----------|-------------|-----------|
| olist_sellers_dataset.csv | 3095 | 4 | 0 | seller_id | Yes - 3095 Distinct and not null values |
| olist_customers_dataset.csv | 99441 | 5 | 0 | customer_id | Yes — 99,441 distinct, but order-scoped (see F5) |


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

**Means :** customer_unique_id repeats within this table while customer_id does not. Whether repeated rows represent repeated purchase depends on row-per-order claim which is unverified (see Q1).

**Action :** Group on customer_unique_id, not customer_id, for any customer-level count or aggregation — the two differ by 3,345 (99,441 vs 96,096).

### F7 - customer_state covers more state codes than seller_state

**Table :** olist_sellers_dataset.csv, olist_customers_dataset.csv.

**Checked :** Compared the set of distinct seller_state values against the set of distinct customer_state values, in both directions.

**Found :** 23 distinct codes in seller_state, 27 in customer_state. Codes appear in customer_state but not seller_state: ['AL', 'AP', 'RR', 'TO']. Appear in seller_state but not customer_state: [].

**Means :** The two columns do not cover the same set of state codes.

**Action :** Build the state/geography dimension from the union of seller_state and customer_state. Add a DQ test asserting every state value in both source tables resolves to a row in that dimension.

## 3.Open Questions

### Q1- Does one row represent one order?

**Table :** olist_customers_dataset.csv.

**Observed :** customer_id is unique per row. Kaggle documentation describes it as order-scoped; this is unverified against the orders table.

**Why deferred :** Needs to profile orders table.

**Resolve by :** after profiling orders