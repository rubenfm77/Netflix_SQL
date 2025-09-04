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
