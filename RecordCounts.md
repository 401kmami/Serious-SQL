# Record Counts & Distinct Values


<p align="center">
  <img width="500" height="500" src="https://img.ifunny.co/images/c9e6531e6ac4cb38f0c9bf554e878e7e0c5785be48694903a0852e5ce43e650d_1.jpg">
</p>

In these notes, we will cover looking at unique and distinct records. We are using the dvd_rentals sample table given in the course

# How many rows are there in the film list table? 

`Select
  Count(*) As row_count
FROM dvd_rentals.film_list
`

997 rows

Tip: Count(1) & Count(*) are interchangeable?

# Unique Column Values
* dedupe, deduplicate, distinct or unique  are interchangable

I want to know the unique values for a given field/column

 SELECT DISTINCT 
    rating
FROM dvd__rentals.film_list

I want to know the COUNT of the distinct values
```sql
SELECT
    COUNT(DISTINCT rating) AS rating_count
FROM dvd_rentals.film_list
```
Now lets say I want to start grouping my counts? 

**GROUP BY Function!**

Entire Code 
```sql
-- Simplified table example
SELECT
  fid,
  title,
  category,
  rating,
  price
FROM dvd_rentals.film_list
LIMIT 10;

-- Group by query using common table expression
WITH example_table AS (
  SELECT
    fid,
    title,
    category,
    rating,
    price
  FROM dvd_rentals.film_list
  LIMIT 10
)
SELECT
  rating,
  COUNT(*) as record_count
FROM example_table
GROUP BY rating
ORDER BY record_count DESC;
```

## **Block 1**
> SELECT
fid,
  title,
  category,
  rating,
  price
FROM dvd_rentals.film_list
LIMIT 10;

This part of the code is condensing the table to 10 rows for these specific columns

## **Block 2**

> -- Group by query using common table expression
WITH example_table AS (
  SELECT
    fid,
    title,
    category,
    rating,
    price
  FROM dvd_rentals.film_list
  LIMIT 10
)

* using WITH command to give simplified table an alias 
* This was we can add on other functions 

## **Block 3**

> SELECT
  rating,
  COUNT(*) as record_count
FROM example_table
GROUP BY rating
ORDER BY record_count DESC;

* adding a count and alias for count of the records
* GROUP BY bc we want the results to be chunked by ratings
* ORDER BY bc we need the sort by the record_count
* We want to see which rating has the highest record_count

Now we are splitting the rows into groups using the GROUP BY function

Note: When using aggregate functions in group by, only 1 row is returned for each group 

 <mark>**COME BACK TO THIS** Only the expression that is used in the GROUP BY grouping elements will be returned along with a single column value for each aggregate function used in the column expressions for the SELECT statement."<mark>

Example Code
```sql
SELECT
  rating,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY rating;
```

> COUNT(*) AS frequency
we are counting the rows for rating and renaming this column as frequency


# Adding a percentage column

* a modified window function combining both SUM and COUNT functions with an OVER() clause

Code Block

```sql
SELECT
  rating,
  COUNT(*) AS frequency,
  COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER () AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC;
```

* `COUNT(*)::NUMERIC` is the numerator
* `NUMERIC` is to avoid the dreaded integer floor division (it will include the decimals)
* `SUM(COUNT(*))` is the denominator
* `OVER` clause defines a window or user-specified set of rows within a query result set [(source)](https://docs.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql?view=sql-server-ver15)

New Code Block to clean up percents
```sql
SELECT
  rating,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC;
```

# Multiple Column Combos

Now we are going to do calculations on a 2+ columns

Code Block
```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY rating, category
ORDER BY frequency DESC
LIMIT 5;
```

In this code block, we are pulling 2 columns along with the frequency calculation column 

# Positional Numbers instead of Column Names

```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY 1,2
```

So in this example, 1 = rating and 2 = category in terms of position of column 

Now when you order by in positional numbers, the game gets complex

```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY 1,2
ORDER BY 2,1,3 DESC
LIMIT 5;
```

In this example, the order of `ORDER BY` is Category (Action being top of the list), rating in alphabetic order then frequency 

example 2 code block

```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 5;
```

In this code block example, we are looking at the top 5 rating and category frequencies since the only column in order by is 3 (frequency) so both rating and category can vary 

example 3 code block
```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY 1,2
ORDER BY frequency DESC
LIMIT 5;
```

This code block has the same output but for orderby, we use the column name frequency instead of positional number 3. This makes it easier to read and understand.

Final example code block

```sql 
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY rating, category
ORDER BY frequency DESC
LIMIT 5;
```

This has the same output as example 2 and 3 code block instead  none of the positional numbers are used. 

# Exercises

### **1. Which actor_id has the most number of unique film_id records in the dvd_rentals.film_actor table?**

```sql
SELECT
  actor_id,
  COUNT(DISTINCT film_id) as unique_records
FROM dvd_rentals.film_actor
GROUP BY actor_id
ORDER BY unique_records DESC;
```

| actor_ID | unique records |
|----------|----------------|
| 107      | 42             |

108=7

### **2. How many distinct fid values are there for the 3rd most common price value in the dvd_rentals.nicer_but_slower_film_list table?**

```sql
SELECT
  price,
  COUNT(DISTINCT fid) AS fid_values
FROM dvd_rentals.nicer_but_slower_film_list
GROUP BY price
ORDER BY fid_values DESC
LIMIT 5;
```

| price | fid_values |
|-------|------------|
| 2.99  |   323      |

323

I mistakenly put price first in the order by when we want the rank by `fid_values`from highest to lowest to see the 3rd most common

### **3. How many unique country_id values exist in the dvd_rentals.city table?**

```sql
SELECT
  COUNT(DISTINCT country_id)
FROM dvd_rentals.city;
```

109

### **4. What percentage of overall total_sales does the Sports category make up in the dvd_rentals.sales_by_film_category table?**

My answer

```sql
SELECT
  category,
  total_sales,
  ROUND(
  100 *
  total_sales::NUMERIC / SUM(total_sales) OVER () ,
  2
  ) AS percentage
FROM dvd_rentals.sales_by_film_category
GROUP BY category, total_sales
ORDER BY percentage DESC;
```

7.88% for sports

The solution

```sql
SELECT
  category,
  ROUND(
    100 * total_sales::NUMERIC / SUM(total_sales) OVER (),
    2
  ) AS percentage
FROM dvd_rentals.sales_by_film_category;
```

Looking at the solution, I can see `total_sales` wasn't need for the second row. I could've also saved time by removing `group by` and `order by`. 

### **5. What percentage of unique fid values are in the Children category in the dvd_rentals.film_list table?**

```sql
SELECT 
  category,
  ROUND(
  100 *
   COUNT(DISTINCT fid)::NUMERIC / SUM(COUNT(DISTINCT fid)) OVER (),
  2
  ) AS percentage
FROM dvd_rentals.film_list
GROUP BY category
ORDER BY category;
```

Children 6.02%

# Appendix

## Division 

```sql
SELECT
    100 / 3 
```

Output:

33 

The output for this won't display the remainder

If the remainder of the output should be a decimal, it will display 0

### Example 1 of Division & NUMERIC

```sql
SELECT
    15 / 20 
```

Output

0

### Example 1 of Division & NUMERIC

```sql
SELECT
    15::NUMERIC / 20 
```

Output

0.7500000000000

NOTE: you can add numeric to numerator and denominator to get decimal/remainder values


