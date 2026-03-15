# Amazon Fashion Product Analysis

## 1. Introduction

This project analyzes a subset of the **Amazon Fashion review dataset** and its corresponding product metadata to understand patterns in product performance, pricing, and seasonal popularity.

The original datasets contained:

| Dataset          | Rows      |
| ---------------- | --------- |
| Product metadata | 826,106   |
| Product reviews  | 2,500,939 |

Both datasets were stored in **compressed JSON Lines format (`.jsonl.gz`)**.

The **reviews dataset** provides:

* review text
* timestamps
* customer feedback

The **metadata dataset** provides:

* product attributes
* price information
* store identifiers

These datasets were processed and combined to enable analysis of relationships between **reviews, price, product descriptions, and popularity**.

---

# Research Questions

### Q1. Product Performance

How is product performance associated with **review length, product description length, and price**?

### Q2. Store Success

Which factors are most strongly associated with **store-level popularity and success**?

### Q3. Seasonal Popularity

How does product popularity change **across seasons**, and is this pattern **consistent across years**?

---

# 2. Data Parsing and Cleaning

## Data Loading

The datasets were provided in **compressed JSON Lines (`.jsonl.gz`) format** and loaded into Python using `pandas`.

```python
pd.read_json(file, lines=True, compression="gzip")
```

Files used:

* `Amazon_Fashion.jsonl.gz` → reviews dataset (`df1`)
* `meta_Amazon_Fashion.jsonl.gz` → metadata dataset (`df2`)

---

## Initial Exploration

Basic inspection methods included:

* `.head()`
* `.info()`
* `.describe()`

Key findings:

| Dataset  | Missing Values |
| -------- | -------------- |
| Reviews  | None           |
| Metadata | Several        |

Major missing values in metadata:

| Column          | Missing Values |
| --------------- | -------------- |
| price           | 775,859        |
| store           | 26,838         |
| bought_together | 826,108        |

---

## Cleaning Steps

The following preprocessing steps were applied:

* Removed **image-related columns** from the reviews dataset
* Removed `images`, `videos`, and `bought_together` columns from metadata
* Removed **51 rows with missing store values**
* Converted **timestamp** from `int64` to `datetime`
* Extracted **month and year** columns
* Dropped `categories` and `main_categories`
* Renamed review title column to **`heading`**
* Joined reviews and metadata using **SQL on `parent_asin`**

The final merged dataset was saved as:

```
sample.csv
```

---

## Subsampling Strategy

Due to dataset size, subsampling was required.

### Reviews dataset

* Selected the **first 5,000 rows**

### Metadata dataset

The first rows could not be used due to many missing price values.

Instead:

* streamed the file line-by-line
* kept rows with **non-null prices**
* collected **5,000 valid records**

---

## Final Dataset Summary

| Feature | Value |
| ------- | ----- |
| Rows    | 9,792 |
| Columns | 18    |

Remaining missing values appear after the join but **do not affect the analysis**.

Outlier handling was deferred to the **EDA stage**.

---

# 3. Relational Database and SQL Analysis

## Database Schema

A relational database was created with two main tables:

| Table  | Rows  | Columns |
| ------ | ----- | ------- |
| review | 5,000 | 11      |
| meta   | 4,949 | 9       |

The cleaned datasets were inserted into the database and renamed:

* `review`
* `meta`

---

## Query 1: Top Products by Rating Volume

### Purpose

Identify **most popular and highly rated products**.

### SQL Techniques

* `JOIN` on `parent_asin`
* `GROUP BY`
* `ORDER BY rating_number`
* `LIMIT 10`

### Key Insights

Top products combine:

* **26,000–36,000 ratings**
* **average ratings between 4.2–4.6**

Multiple ASINs belonging to the same product often appear together, suggesting strong engagement and sustained popularity.

This supports **Research Questions 1 and 3**.

---

## Query 2: Best-Performing Products by Time

### Purpose

Analyze how popularity changes **across months and years**.

### SQL Techniques

* `JOIN`
* Window function:

```sql
ROW_NUMBER() OVER (PARTITION BY month/year ORDER BY rating_number DESC)
```

### Key Insights

Some products consistently dominate:

* **Russell Athletic hoodie**
* **NELEUS compression tops**

These items perform strongly across many months.

Other products (dresses, joggers, sunglasses) peak during specific seasons.

This indicates:

* some products have **year-round demand**
* others have **seasonal popularity**

---

## Query 3: High- vs Low-Priced Verified Products

### Purpose

Evaluate the **relationship between price and perceived value**.

### SQL Techniques

* Filter `verified_purchase = 1`
* Order by price ascending and descending
* Limit to top results

### Key Insights

Both **high-priced** and **low-priced** products receive strong ratings.

Examples:

| Product Type    | Pattern                               |
| --------------- | ------------------------------------- |
| Expensive items | strong ratings but fewer reviews      |
| Cheap items     | strong ratings and high review volume |

This suggests **high perceived value can exist at many price levels**.

---

# 4. Exploratory Data Analysis

## Correlation Analysis

A correlation heatmap examined relationships between:

* review length
* description length
* price
* rating count

Key findings:

* Longer descriptions slightly correlate with more ratings.
* Longer reviews appear more frequently on popular products.
* Price has a modest positive relationship with rating counts.

However:

**average rating remains stable regardless of these factors.**

---

## Price vs Rating

Scatter plots show:

* both **cheap and expensive items** appear across the **3–5 star rating range**
* high popularity products cluster around **4–5 stars**

This suggests:

* price does not guarantee higher ratings
* highly rated products often attract many reviews.

---

## Store-Level Popularity

Analysis of store performance shows:

* higher prices often correlate with **slightly fewer ratings**
* stores with higher ratings tend to attract **more reviews**

Successful stores appear to balance:

* reasonable pricing
* strong ratings
* consistent customer engagement

---

## Seasonal Trends

Monthly rating counts reveal **seasonal spikes**.

Notable peaks occur around:

* **May**
* **October**
* **November**

This likely reflects:

* spring fashion demand
* holiday shopping periods

While seasonal effects are visible, **exact peaks vary across years**, indicating fluctuating demand patterns.

---

# 5. Limitations and Reflection

## Dataset Limitations

Some variables could not be used due to data issues:

* `bought_together` was missing
* `category` and `main_category` had identical values

Large compressed datasets also slowed processing.

The **data cleaning and merging process** was the most time-consuming step.

---

## Learning Outcomes

This project strengthened skills in:

* data cleaning
* relational database design
* SQL querying
* exploratory data analysis
* interpreting large-scale e-commerce datasets

Future work could explore:

* category-level behavior
* deeper customer segmentation
* predictive modeling for product success.

---

