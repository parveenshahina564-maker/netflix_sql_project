# Netflix Movies and TV Shows Data Analysis using SQL

|[Netflix Logo](https://github.com/parveenshahina564-maker/netflix_sql_project/blob/main/image%20netflix.png)

## Overview

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the datasets. the following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

. Analyze the distribution of content types (movies vs TV Shows).
. Identify the most common ratings for movies and TV Shows.
. Lists and analyze content based on release years, countries, and durations.
. Explore and categorize content based on specific criteria and keywords.

### Dataset

The data for this project is sourced from Kaggle dataset:
. Dataset link: [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows)

## Schema

```sql
-- Netflix Project
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id VARCHAR(6),
	type    VARCHAR(10),
    title   VARCHAR(150),
    director	VARCHAR(208),
    casts	VARCHAR(1000),
    country	VARCHAR(150),
    date_added		VARCHAR(50),
    release_year	INT,
    rating	VARCHAR(10),
    duration   VARCHAR(15),
    listed_in	VARCHAR(100),
    description VARCHAR(250)
);
```

## Business Problems and Solution

### 1. Count the number of Movies vs TV Shows

```sql
SELECT 
    type,
	COUNT(*) as total_content
FROM netflix
GROUP BY type
```

### 2. Find the most common rating for movies and TV Shows

```sql
SELECT
    type,
	rating
FROM

(
    SELECT
       type,
	   rating,
	   COUNT(*),
       RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
    FROM netflix
    GROUP BY 1, 2
) as t1
WHERE
    ranking = 1
```

###  3. List all movies released in a specific year (e.g., 2020)

```sql
-- filter 2020
-- movies
SELECT * FROM netflix
WHERE 
    type = 'Movie'
	AND
	release_year = 2020
  ```

  ### 4. Find the top 5 counties with most content on Netflix

  ```sql
  SELECT
    UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
	 COUNT(show_id) as total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```

### 5. Identify the longest movie?

```sql
SELECT * FROM netflix
WHERE
    type = 'Movie'
	AND
	duration = (SELECT MAX(duration) FROM netflix)
```

### 6.  Find content added in the last 5 years

```sql
SELECT 
    *
	FROM netflix
WHERE
    TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 year'
SELECT CURRENT_DATE - INTERVAL '5 year'
```

### 7. Find all the movies/TV shows by director 'Rajiv Chilaka'

```sql
SELECT * FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%'
```

### 8. List all the TV shows with more than 5 seasons

```sql
SELECT  
    *
	FROM netflix
WHERE
    type = 'TV Show' 
	AND
	SPLIT_PART(duration,' ', 1)::numeric > 5
```

### 9. Count the number of content items in eaach genre

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY( listed_in, ',')) as genre,
	COUNT(show_id) as total_content
FROM netflix
GROUP BY 1
```

### 10. Find each year and average number of content release by India on Netflix. 
return top 5 year with highest avg content release

```sql
total content 333/972

SELECT 
    EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) as year,
    COUNT(*) as yearly_content, 
	ROUND(
	COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country = 'India')::numeric * 100 
	,2)as avg_content_per_year
FROM netflix
WHERE country = 'India'
GROUP BY 1
```

### 11.  List all movies thata are documentaries

```sql
SELECT * FROM netflix
WHERE
    listed_in ILIKE '%documentaries%'
```

### 12. Find all content without a director

```sql
SELECT * FROM netflix
WHERE
    director IS NULL
```

  ### 13. Find how many movies actor 'Salman Khan' appeared in last 10 years

  ```sql
  SELECT * FROM netflix
WHERE
   casts ILIKE '%Salman Khan%'
   AND
   release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
   ```

   ### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India

   ```sql
    SELECT 
 UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
 COUNT(*) as total_content
 FROM netflix
 WHERE country ILIKE '%India%'
 GROUP BY 1
 ORDER BY 2 DESC
 LIMIT 10
 ```

 ### 15. Categorize the content based on the presence of the keywords 'kill' and 'voilence' 
in the description field. label content containing these keywords as 'Bad' and all 
other content as 'Good'. count how many item falls intpo each category.

```sql
WITH new_table
AS
(
SELECT 
*, 
    CASE
	WHEN 
	     description ILIKE '%kill%' OR
	     description ILIKE '%violence%' THEN 'Bad_content'
         ELSE 'Good_content'
	END category	 
FROM netflix
)
SELECT 
     category,
	 COUNT(*) as total_content
FROM new_table
GROUP BY 1
```
