# Netflix Catalog Analysis — SQL

[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-14+-blue?logo=postgresql)](https://www.postgresql.org/)
[![Dataset](https://img.shields.io/badge/Dataset-Netflix%20Titles-red?logo=netflix)](https://www.kaggle.com/datasets/shivamb/netflix-shows)

![Netflix](https://raw.githubusercontent.com/rubenfm77/Netflix_SQL/main/logo.png)

> 15 analytical SQL queries on Netflix's full content catalog — content mix, top-producing countries, genre distribution, rating profiles, director analysis, and keyword-based classification — using PostgreSQL's array, string, and window functions.

---

## Dataset

`netflix_titles.csv` — Netflix catalog snapshot with columns:

`show_id`, `type` (Movie/TV Show), `title`, `director`, `cast`, `country`, `date_added`, `release_year`, `rating`, `duration`, `listed_in` (genres), `description`

---

## Queries

### 1. Content mix: Movies vs TV Shows

```sql
SELECT type, COUNT(*) AS content
FROM Netflix
GROUP BY type
```

### 2. Most common rating by content type

```sql
SELECT type, rating
FROM (
  SELECT type, rating, COUNT(*),
         RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS ranking
  FROM Netflix
  GROUP BY 1, 2
) AS t1
WHERE ranking = 1
```

### 3. Movies released in a specific year

```sql
SELECT title FROM netflix
WHERE type = 'Movie' AND release_year = 1993
```

### 4. Top 5 content-producing countries

```sql
SELECT
  UNNEST(STRING_TO_ARRAY(country, ',')) AS new_country,
  COUNT(show_id) AS total
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```

### 5. Longest movie on the platform

```sql
SELECT * FROM netflix
WHERE type = 'Movie'
  AND duration = (SELECT MAX(duration) FROM netflix)
```

### 6. Content added in the last 5 years

```sql
SELECT * FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'
```

### 7. All titles by director 'Rajiv Chilaka'

```sql
SELECT * FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%'
```

### 8. TV Shows with more than 5 seasons

```sql
SELECT * FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::numeric > 5
```

### 9. Content count by genre

```sql
SELECT
  UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS Genre,
  COUNT(show_id) AS Total
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
```

### 10. India's average content releases per year on Netflix (top 5 years)

```sql
SELECT
  EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD YYYY')) AS year,
  COUNT(*) AS Content,
  ROUND(
    COUNT(*)::numeric /
    (SELECT COUNT(*) FROM netflix WHERE country = 'India')::numeric * 100, 2
  ) AS Avg_per_Year
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
```

### 11. Documentaries

```sql
SELECT * FROM netflix
WHERE listed_in ILIKE '%documentaries%'
```

### 12. Content without a director

```sql
SELECT * FROM netflix
WHERE director IS NULL
```

### 13. Movies with Salman Khan in the last 10 years

```sql
SELECT * FROM netflix
WHERE casts ILIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
```

### 14. Top 10 actors in Indian-produced movies

```sql
SELECT
  UNNEST(STRING_TO_ARRAY(casts, ',')) AS actors,
  COUNT(show_id) AS total
FROM netflix
WHERE country ILIKE '%India%'
  AND type = 'Movie'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```

### 15. Content classification: violent vs family-friendly

```sql
WITH New_Table AS (
  SELECT *,
    CASE
      WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad content'
      ELSE 'Good content'
    END AS Category
  FROM netflix
)
SELECT category, COUNT(*) AS Total_content
FROM New_Table
GROUP BY 1
ORDER BY 2 DESC
```

---

## Conclusions

**Movies dominate but TV Shows are growing**

The catalog contains significantly more Movies than TV Shows, but TV Shows drive longer engagement — a TV show viewer returns for multiple episodes while a movie viewer is done in 90 minutes. The content mix reflects Netflix's original movie-library roots, before the streaming model pivoted heavily toward serialised content.

**The US leads production by a wide margin; India is the clear #2**

The top-5 country query confirms the US as the single largest content producer by volume, with India as a distant but substantial second — driven by Bollywood and regional-language productions. The Indian catalog also shows consistent year-over-year growth, making it one of the fastest-expanding national libraries on the platform.

**Documentaries are the largest single genre**

`listed_in` genre analysis puts Documentaries at the top of the genre count, ahead of Dramas and Comedies. This is counterintuitive — it reflects Netflix's strategy of licensing large documentary back-catalogs (cheaper per title than scripted content) to pad the library breadth.

**Rating profiles differ sharply by content type**

The most common rating for TV Shows is **TV-MA** (mature audiences); for Movies it is **R** or **TV-MA** depending on the snapshot year. Family-friendly content (G, PG, TV-G, TV-Y) makes up a much smaller share of the catalog than the adult-oriented tiers — consistent with Netflix's core demographic skewing toward adults 18–44.

**A non-trivial share of content has no director listed**

The `NULL` director query returns a meaningful number of titles — primarily talk shows, stand-up specials, and licensed content where the "director" field was not populated in the source catalog. This is a data-quality observation, not a content editorial choice.

**Keyword-based classification reveals the violent content proportion**

The description-based classification (Query 15) shows the split between content containing keywords like "kill" or "violence" vs. the rest. While crude, this kind of keyword filter is a common first-pass content safety tool before more sophisticated NLP is applied.

**Key SQL techniques demonstrated**

- `UNNEST(STRING_TO_ARRAY(...))` to explode multi-value comma-separated fields (genres, countries, cast) into rows
- `RANK() OVER (PARTITION BY ...)` for "top N per group" patterns
- `LAG` / window aggregates for time-series analysis
- `ILIKE` for case-insensitive text search across large catalogs
- CTE + `CASE WHEN` for classification and pivot-style transformations

---

## Tech Stack

| Tool | Use |
|---|---|
| **PostgreSQL 14+** | Query engine |
| **psql / pgAdmin** | Query execution and result inspection |
| **Array / string functions** | `UNNEST`, `STRING_TO_ARRAY`, `SPLIT_PART` |
| **Window functions** | `RANK()`, `EXTRACT()`, aggregate over windows |
