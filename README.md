# apple_sales_analysis
A hands-on SQL project analyzing Apple retail sales, stores, products, and warranty data through 15 business problems.
# 🍎 Apple Retail Sales SQL Project

## About the Project

This project is based on an **Apple retail sales dataset** containing around **50K+ rows** of sales and business data.

I worked with five tables:

* `stores`
* `category`
* `products`
* `sales`
* `warranty`

The main goal of this project is to practice SQL by solving business-related questions using real sales, product, store, and warranty data.

Instead of only writing basic `SELECT` statements, I used different SQL concepts such as **JOINs, GROUP BY, aggregate functions, subqueries, ranking functions, and date functions**.

---

## 🗃️ Database Tables

### `stores`

Contains information about Apple stores.

| Column       | Description            |
| ------------ | ---------------------- |
| `store_id`   | Unique ID of the store |
| `store_name` | Name of the store      |
| `city`       | City of the store      |
| `country`    | Country of the store   |

### `category`

Contains product category information.

| Column          | Description          |
| --------------- | -------------------- |
| `category_id`   | Unique category ID   |
| `category_name` | Name of the category |

### `products`

Contains information about Apple products.

| Column         | Description         |
| -------------- | ------------------- |
| `product_id`   | Unique product ID   |
| `product_name` | Product name        |
| `category_id`  | Product category    |
| `launch_date`  | Product launch date |
| `price`        | Product price       |

### `sales`

Contains individual sales transactions.

| Column       | Description                   |
| ------------ | ----------------------------- |
| `sale_id`    | Unique sale ID                |
| `sale_date`  | Date of sale                  |
| `store_id`   | Store where the sale happened |
| `product_id` | Product sold                  |
| `quantity`   | Number of units sold          |

### `warranty`

Contains warranty claim information.

| Column          | Description                  |
| --------------- | ---------------------------- |
| `claim_id`      | Unique warranty claim ID     |
| `claim_date`    | Date of warranty claim       |
| `sale_id`       | Related sale                 |
| `repair_status` | Status of the warranty claim |

---

# 📊 Business Problems

I solved **15 SQL business problems**, starting with store and sales analysis and gradually moving towards more complex joins, subqueries, and window functions.

---

## 🟢 Q1. Find the number of stores in each country

The first question looks at the distribution of Apple stores across different countries.

```sql
select country
,count(store_id) as total_stores
 from stores GROUP BY country
 ORDER BY 2 desc;
```

**SQL concepts used:**

* `COUNT()`
* `GROUP BY`
* `ORDER BY`

---

## 🟢 Q2. Calculate the number of units sold by each store

Here I joined the `sales` and `stores` tables to find how many total units were sold by each store.

```sql
select 
       s.store_id,st.store_name,
       sum(s.quantity) as total_unit_sold
    from sales as s
    join stores as st 
    on s.store_id=st.store_id
     group by 1,2
     Order by 3;
```

**SQL concepts used:**

* `JOIN`
* `SUM()`
* `GROUP BY`
* Column positions in `GROUP BY` and `ORDER BY`

---

## 🟢 Q3. Identify how many sales occurred in December 2023

This query checks the number of sales recorded during the specified 2023 period.

```sql
SELECT count(sale_id)
 from sales
 where sale_date
 BETWEEN
   '2023-01-01'
and
   '2023-12-31';
```

**SQL concepts used:**

* `COUNT()`
* `BETWEEN`
* Date filtering

---

## 🟢 Q4. Determine how many stores have had a warranty claim filed

This query connects warranty claims with sales and then identifies the stores associated with those claims.

```sql
select * from stores
WHERE store_id    in (
            select DISTINCT store_id from sales as s
            RIGHT join  warranty as W
            on s.sale_id=w.sale_id
);
```

**SQL concepts used:**

* Subquery
* `DISTINCT`
* `RIGHT JOIN`
* `IN`

---

## 🟢 Q5. Calculate the percentage of warranty claims marked as "Warranty Void"

Here I calculated what percentage of all warranty claims were marked as warranty void.

```sql
select
ROUND
  (count(claim_id)/
                (select count(*) from warranty
)*100,2)
        as warranty_void_percentage

from warranty
 where repair_status="warranty void";
```

**SQL concepts used:**

* Subquery
* `COUNT()`
* `ROUND()`
* Percentage calculation

---

## 🟡 Q6. Identify which store had the highest total units sold in the last 2 years

This query looks at recent sales and compares the total number of units sold by each store.

```sql
select
   s.store_id,
   st.store_name,
      sum(quantity) as total_sales
      from sales as s
        join stores as st
            on s.store_id=st.store_id
            where s.sale_date>= CURRENT_DATE - INTERVAL 2 YEAR
      group by s.store_id,store_name
      order by total_sales
      LIMIT 1;
```

**SQL concepts used:**

* `JOIN`
* `SUM()`
* Date interval
* `GROUP BY`
* `LIMIT`

---

## 🟡 Q7. Count the number of unique products sold in the last year

This query uses `DISTINCT` to count different products appearing in the sales data.

```sql
select 
  count(DISTINCT s.product_id) as total_product
 from sales as s
     where s.sale_date>= CURRENT_DATE - INTERVAL 2 YEAR;
```

**SQL concepts used:**

* `COUNT()`
* `DISTINCT`
* Date interval

---

## 🟡 Q8. Find the average price of products in each category

Here I joined products with categories and calculated the average product price for every category.

```sql
select
pt.category_id,
ct.category_name,
     ROUND(avg(price),0) as average_price
      from products as pt
      join category as Ct
           on pt.category_id=ct.category_id
           group by pt.category_id,2
           order by 3 desc;
```

**SQL concepts used:**

* `JOIN`
* `AVG()`
* `ROUND()`
* `GROUP BY`
* `ORDER BY`

---

## 🟡 Q9. How many warranty claims were filed in 2024?

This query filters warranty claims by year and counts the total claims.

```sql
select 
 count(*) as total_claim_warranty
 from warranty
 where EXTRACT(YEAR from claim_date)=2024;
```

**SQL concepts used:**

* `COUNT()`
* `EXTRACT()`
* Date analysis

---

## 🟡 Q10. For each store, identify the best-selling day based on highest quantity sold

This was one of the more interesting queries because I used a **window function** to rank the selling days for each store.

```sql
select * from (
               select 
               store_id,
               dayname(sale_date) as daname,
               sum(quantity) as total_unit_sold,
               rank() over (partition by store_id order by sum(quantity) desc) as ranks
               from sales 
               group by 1,2) as t1

where ranks=1;
```

The `RANK()` function helps identify the highest-selling day separately for every store.

**SQL concepts used:**

* Subquery
* `DAYNAME()`
* `SUM()`
* `RANK()`
* Window functions
* `PARTITION BY`

---

# 🔵 More Advanced Business Problems

## Q11. Identify the least-selling product in each country

This query combines sales, stores, and products to find the product with the lowest total sales in each country.

```sql
select * from
         (select st.country,
         ps.product_name,
         sum(quantity) as total_unit_sales,
         RANK() OVER (PARTITION BY ST.country ORDER BY sum(quantity)) AS RANKS 
         from sales as S
                join stores as st
                           on s.store_id=st.store_id
                join products as Ps
                          on s.product_id=ps.product_id
                              GROUP BY 1,2) as t1

    where  ranks=1;
```

The important part here is the use of `RANK()` with `PARTITION BY country`.

**SQL concepts used:**

* Multiple `JOIN`s
* `SUM()`
* Subquery
* Window function
* `RANK()`
* `PARTITION BY`

---

## Q12. Calculate how many warranty claims were filed within 180 days of a product sale

Here I compared the warranty claim date with the original sale date.

```sql
SELECT count(*)
   from warranty as W 
   left join sales AS s
            on w.sale_id=s.sale_id
                     where w.claim_date-s.sale_date<=180;
```

This helps understand how many claims happened relatively soon after a purchase.

**SQL concepts used:**

* `LEFT JOIN`
* Date difference
* `COUNT()`

---

## Q13. Determine how many warranty claims were filed for products launched in the last 3 years

This query connects warranty claims, sales, and products to check warranty activity for recently launched products.

```sql
SELECT 
pt.product_name,
count(w.claim_id) as nu_claim,
count(s.sale_id) as sale_id

from warranty as w
right join sales AS S
on w.sale_id=s.sale_id

join products as pt
on s.product_id=pt.product_id

where pt.launch_date>= current_date - INTERVAL 3 YEAR

group by 1

having nu_claim>0;
```

The `HAVING` condition makes sure that only products with warranty claims are included.

**SQL concepts used:**

* Multiple `JOIN`s
* `COUNT()`
* Date interval
* `GROUP BY`
* `HAVING`

---

## Q14. List the months in the last 3 years where sales exceeded 700 units in the USA

This query looks at USA sales month by month and keeps only the months where sales crossed the required threshold.

```sql
select
DATE_FORMAT(sale_date,'%m-%Y') as month_year,
sum(quantity) as total_unit_sold

from sales as s
join stores as st
on s.store_id=st.store_id

WHERE country='usa'
and sale_date>=CURRENT_DATE-INTERVAL 3 YEAR

GROUP BY 1

having sum(quantity)>=700;
```

This is useful for finding months with comparatively high sales activity.

**SQL concepts used:**

* `JOIN`
* `DATE_FORMAT()`
* `SUM()`
* `GROUP BY`
* `HAVING`
* Date interval

---

## Q15. Identify the product category with the most warranty claims filed in the last 2 years

For the final question, I connected four tables:

`warranty → sales → products → category`

This allows us to see which product category has generated the most warranty claims recently.

```sql
select 
category_name,
count(claim_id) as total_claim

from warranty as W

left join sales as s
on w.sale_id=s.sale_id

join products as pt
on s.product_id=pt.product_id

join category as ct
on pt.category_id=ct.category_id

where w.claim_date>= CURRENT_DATE-INTERVAL 2 year

group by 1

order by 2 desc;
```

**SQL concepts used:**

* Multiple table joins
* `COUNT()`
* `GROUP BY`
* `ORDER BY`
* Date filtering

---

# 🧠 What I Practiced

Through these 15 problems, I practiced several important SQL concepts:

### Basic SQL

* `SELECT`
* `WHERE`
* `GROUP BY`
* `ORDER BY`
* `HAVING`
* `DISTINCT`

### Aggregate Functions

* `COUNT()`
* `SUM()`
* `AVG()`
* `ROUND()`

### Joins

* `JOIN`
* `LEFT JOIN`
* `RIGHT JOIN`

### Subqueries

* Nested `SELECT`
* `IN`
* Filtering using subqueries

### Window Functions

* `RANK()`
* `PARTITION BY`

### Date Functions

* `CURRENT_DATE`
* `INTERVAL`
* `EXTRACT()`
* `DATE_FORMAT()`
* `DAYNAME()`

---

# 📈 What This Project Helped Me Understand

While working on this project, I got practical experience with questions that are common in real data analysis work.

For example:

* How many stores are operating in each country?
* Which stores are selling the most units?
* Which products are being sold the least?
* How many warranty claims are being generated?
* Which product categories have more warranty issues?
* How can sales be compared across different time periods?
* How can window functions be used to rank data within groups?

The main learning for me was not just writing the SQL syntax, but understanding **how to convert a business question into a SQL query**.

---

# 🛠️ SQL Skills Used

```text
SQL
│
├── Filtering
├── Aggregation
├── GROUP BY
├── HAVING
├── ORDER BY
├── DISTINCT
│
├── JOINs
│   ├── INNER JOIN
│   ├── LEFT JOIN
│   └── RIGHT JOIN
│
├── Subqueries
│
├── Window Functions
│   └── RANK()
│
└── Date & Time Analysis
    ├── INTERVAL
    ├── EXTRACT()
    ├── DATE_FORMAT()
    └── DAYNAME()
```

---

# 📂 Project Structure

```text
apple-retail-sales-sql/
│
├── README.md
│
├── SQL/
│   └── apple_sales_queries.sql
│
└── Dataset/
    ├── category
    ├── products
    ├── sales
    ├── stores
    └── warranty
```

---

# 🚀 Final Thoughts

This project gave me a good opportunity to practice SQL on a retail dataset instead of working only with small examples.

The questions gradually move from simple aggregation to **multiple-table joins, subqueries, date analysis, and window functions**.

I also got more comfortable with looking at a business problem first and then deciding which SQL concepts are needed to solve it.

Overall, this project helped me improve my understanding of **SQL-based data analysis and business problem solving**.
