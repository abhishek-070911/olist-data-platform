# Data Profiling Report — Olist Brazilian E-Commerce

Source: [Kaggle, olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

Profiled by: Abhishek Patra

Last updated: 2026-09-12

## 1.Inventory

| File | Rows | Cols | Null Cols | Primary Key | PK Valid? |
|------|------|------|-----------|-------------|-----------|
|olist_sellers_dataset.csv | 3095 | 4 | 0 | seller_id | Yes - 3095 Distinct and not null |


### Column cardinality — sellers
| Column | Distinct | Note |
|--------|----------|------|
| seller_id | 3095 | = Row count |
| seller_zip_code_prefix | 2246 | not unique per seller |
| seller_city | 611 | includes variant spellings (see F4) |
| seller_state | 23 | distinct codes present; includes DF (Federal District) | 


## 2.Findings

### F1 - seller_zip_code_prefix is not unique per seller

**Table :** olist_sellers_dataset.csv.

**Checked :** Distinct 'seller_zip_code_prefix' vs row count.

**Found :** 2246 distinct values across 3095 sellers.

**Means :** Multiple sellers share same zip code,so it identifies 'location' and not a seller.

**Action :** Treat it as descriptive attribute,not as a key.

### F2 - Seller base is heavy at SP state

**Table :** olist_sellers_dataset.csv.

**Checked :** Counted sellers per state (value_counts on seller_state).

**Found :** SP holds 1,849 of 3,095 sellers (60%); top 3 (SP, PR, MG) hold 2,442 (79%).

**Means :** Large number of sellers are in SP compared to other states.

**Action :** None for the model. Recorded as context: 20 states have fewer than 50 sellers each, so orders shipping there are largely interstate — delivery-time analysis must control for this.

### F3 - Zip is parsed as int64,stripping leads zero

**Table :** olist_sellers_dataset.csv.

**Checked :** Counted rows where prefix is shorter than 5 characters.

**Found :** 1,027 rows (33%) hold 4-character values. Lengths are only 4 or 5, so every 4-character value is a 5-digit prefix whose leading zero was dropped on read.

**Means :** The source CSV is correct — it contains "04195". The zero was destroyed by pandas inferring int64 on read. This is a defect in our ingestion, not in the source data.

**Action :** Read zip prefix columns with dtype=str at ingestion — in sellers, customers and geolocation alike, so all three join on the same type. Add a DQ test asserting length == 5.

### F4 - seller_city contains variant spellings of the same city

**Table :** olist_sellers_dataset.csv.

**Checked :** Filtered seller_city for 'do rio preto'.

**Found :** 'sao jose do rio preto' (33) and 's jose do rio preto' (1) — the same city under two values.

**Means :** Any group-by on raw city splits this city in two. Other abbreviations likely exist across the 611 values.

**Action :** Do not use raw city as a grouping key. Standardise in Silver against a municipality reference list. Extent of the problem across all 611 values not yet measured.

## 3.Open Questions

### Q1-Do customers span more than sellers?

**Table :** olist_sellers_dataset.csv

**Observed :** Sellers cover 23 states

**Why deferred :** Customers table not profiled

**Resolve by :** after profiling customers