# Netflix Movies and TV Shows Data Analysis using SQL

![Netflix](https://github.com/rubenfm77/Netflix_SQL/blob/main/logo.png)

## Objective

### -- 1. Count Number of Movies vs TV Shows

```sql
SELECT type, COUNT (*) as content
FROM Netflix
GROUP BY type
```
