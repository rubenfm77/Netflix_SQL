# Netflix Movies and TV Shows Data Analysis using SQL

![Netflix](https://github.com/rubenfm77/Netflix_SQL/blob/main/logo.png)

## Objective

The idea is to do some simple SQL queries answering some questions. Check out the original work from Najirh

### 1. Count Number of Movies vs TV Shows

```sql
SELECT type, COUNT (*) as content
FROM Netflix
GROUP BY type
```

### 2. Find the most common rating for Movies and TV Shows

```sql
SELECT 
	type, 
	rating
FROM

(SELECT 
	type, 
	rating, 
	COUNT (*),
	RANK() OVER(PARTITION BY type ORDER BY COUNT(*)DESC) as ranking
FROM Netflix
GROUP BY 1,2) AS t1
WHERE ranking = 1
```

### 3. List all movies released in a specific year.

```sql
SELECT title
FROM netflix
WHERE type ='Movie' AND release_year = 1993
```
### 4. Find top 5 countries with the most content on Netflix

```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
	COUNT (show_id) as total
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```
### 5. Identify the longest movie

```sql
SELECT * FROM netflix
WHERE 
	type = 'Movie'
	AND 
	duration = (SELECT MAX (duration) FROM netflix)
```
### 6. Find the contend added in the last five years

```sql
SELECT 
	*
FROM netflix
WHERE 
	TO_DATE(date_added, 'Month DD, YYYY')>= CURRENT_DATE - INTERVAL '5 years'
```
### 7. Find all the movies/TV shows by director 'Rajiv Chilaka'

```sql
SELECT 
	*
FROM netflix
WHERE 
	director ILIKE '%Rajiv Chilaka%'
```

### 8. List all TV shows with more than 5 seasons
```sql
SELECT 
	*
FROM netflix
WHERE 
	type = 'TV Show'
	AND
	SPLIT_PART(duration, ' ', 1)::numeric > 5 
```
### 9. Count the number of content items in each genre
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS Genre,
	count(show_id) AS Total
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
```
### 10. Find each year and the average numbers of content released by India on Netflix. Return top 5 year with highest avg content release

```sql
SELECT 
	EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD YYYY')) as year,
	COUNT(*) AS Content,
	ROUND(
	COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country = 'India')::numeric *  100 
	,2) AS Avg_per_Year
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
```
#### 11. List all the movies that are documentaries

```sql
SELECT 
	*
FROM netflix
WHERE 
	listed_in ILIKE '%documentaries%'
```

### 12. Find all the content without Director

```sql
SELECT 
	*
FROM netflix
WHERE 
	director IS NULL
```

### 3. Find how many movies the actor 'Salman Khan' appeared in the las 10 years.

```sql
SELECT 
	*
FROM netflix
WHERE 
	casts ILIKE '%Salman Khan%'
	AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) -10
```

### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India

```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
	COUNT (show_id) as total
FROM netflix
WHERE country ILIKE '%India%'
	AND type = 'Movie'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```

### 15. Categorize the content based on the presence of keywords 'kill' and 'violence' in the description field, label content containing keywords as 'bad' and all other content as 'Good'. Count how many items fall into each category

```sql
WITH New_Table AS
(
SELECT *,
	CASE 
	WHEN description ILIKE '%kill%' OR
		description ILIKE '%violence%' THEN 'Bad content'
		ELSE 'Good content'
	END Category
FROM netflix
)
SELECT 
	category,
	COUNT(*) as Total_content

```
FROM New_Table
GROUP BY 1
ORDER BY 2 DESC
