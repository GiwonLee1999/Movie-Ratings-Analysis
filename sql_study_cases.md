# SQL Study Cases - BigQuery Movies Dataset

## **Basic Queries**

### 1️⃣ Retrieve the **top 10 highest-rated movies** (based on `IMDB_Rating`).
```sql
-- Write your SQL answer here
SELECT Series_Title 
FROM `imdb_movies_1000` 
ORDER BY Meta_score
LIMIT 10;
```

### 2️⃣ Find the **number of movies per decade** using `Released_Year`.
```sql
-- Write your SQL answer here
SELECT 
    (FLOOR(Released_Year / 10) * 10) AS Decade,
    COUNT(*) AS Movie_Count
FROM `imdb_movies_1000` 
GROUP BY Decade
order by Decade
```

### 3️⃣ List all movies where **IMDB rating is above 8.5** and `Meta_score` is above 80.
```sql
-- Write your SQL answer here
SELECT 
 Series_Title
FROM `imdb_movies_1000` 
WHERE IMDB_Rating > 8.5 AND Meta_score > 80
```

### 4️⃣ Find the **most common movie certificate** (e.g., PG, R, etc.).
```sql
-- Write your SQL answer here
SELECT 

 Certificate,
 Count(Certificate) as count_certificate,
 ROW_NUMBER() over(order by count(certificate) desc) as ranking_certificate


FROM `imdb_movies_1000` 


group by certificate

```

### 5️⃣ Identify the **oldest and newest movies** in the dataset.
```sql
-- Write your SQL answer here
-- Returns multiple of the movies because there are multiple movies in max year
SELECT 
    'Oldest' AS Category, Series_Title, Released_Year 
FROM `imdb_movies_1000` 
WHERE Released_Year = (SELECT MIN(Released_Year) FROM `imdb_movies_1000` )

UNION ALL

SELECT 
    'Newest' AS Category, Series_Title, Released_Year 
FROM `imdb_movies_1000` 
WHERE Released_Year = (SELECT MAX(Released_Year) FROM `imdb_movies_1000` );


```

---

## **Aggregation & Grouping**

### 6️⃣ Find the **average IMDB rating** of movies released in each decade.
```sql
-- Write your SQL answer here

WITH cte AS (
  SELECT 

  (FLOOR(Released_Year/10)) * 10 AS Decade,
  ROUND(AVG(IMDB_Rating),2) AS Rating,
  COUNT(*) AS num_movie_released
  FROM `imdb_movies_1000` 
  GROUP BY Decade
  ORDER BY Decade 
)
SELECT 

  *,
  ROW_NUMBER() OVER(ORDER BY Rating DESC) AS rate_ranking

FROM cte
```

### 7️⃣ Calculate the **total and average Gross revenue** of movies per genre.
```sql
-- Write your SQL answer here
SELECT 

  Genre,
  SUM(Gross) AS Total_Gross,
  ROUND(AVG(Gross),2) AS Average_Gross,
  ROW_NUMBER() OVER(ORDER BY SUM(Gross) DESC) AS gross_ranking

FROM `imdb_movies_1000` 
GROUP BY Genre

```

### 8️⃣ Find the **top 5 directors** with the highest average IMDB rating.
```sql
-- Write your SQL answer here
WITH cte AS(
SELECT 

  Director,
  AVG(IMDB_Rating) AS avg_rating,
  ROW_NUMBER() OVER(ORDER BY AVG(IMDB_Rating) DESC) AS rate_ranking


FROM `imdb_movies_1000` 
GROUP BY Director
)
SELECT * from cte
WHERE rate_ranking < 6

```

### 9️⃣ Count how many movies each **actor (`Star1`) has starred in**.
```sql
-- Write your SQL answer here
SELECT 

  Star1 AS Actor,
  COUNT(*) AS Movie_Count

FROM `imdb_movies_1000` 
Group by Star1
ORDER BY Movie_Count DESC
```

### 🔟 Identify the **genres with the highest average Meta Score**.
```sql
-- Write your SQL answer here
SELECT 
 Genre,
 ROUND(AVG(Meta_score),2) AS Highest_AVG_Meta_Score
FROM `imdb_movies_1000` 
Group by Genre
ORDER BY Highest_AVG_Meta_Score DESC
LIMIT 5
```

---

## **Advanced Queries & Joins**

### 1️⃣1️⃣ Find the **top-grossing movie for each year**.
```sql
-- Write your SQL answer here
SELECT Series_Title, Released_Year, Gross
FROM (
    SELECT 
        Series_Title, 
        Released_Year, 
        Gross,
        ROW_NUMBER() OVER (PARTITION BY Released_Year ORDER BY Gross DESC) AS Rank
    FROM `imdb_movies_1000` 
    WHERE Gross IS NOT NULL
) RankedMovies
WHERE Rank = 1
ORDER BY Released_Year

```

### 1️⃣2️⃣ Identify movies where **Meta Score is missing**, and check their average IMDB rating.
```sql
-- Write your SQL answer here
SELECT 

Series_Title,
AVG(IMDB_Rating)

FROM `imdb_movies_1000` 
WHERE Meta_score is NULL OR Meta_score = 0  
GROUP BY Series_Title
```

### 1️⃣3️⃣ Find the **highest-grossing movie per director**.
```sql
-- Write your SQL answer here
SELECT 
    Director, 
    Series_Title, 
    Gross
FROM (
    SELECT 
        Director, 
        Series_Title, 
        Gross,
        ROW_NUMBER() OVER (PARTITION BY Director ORDER BY Gross DESC) AS Rank
    FROM `imdb_movies_1000` 
    WHERE Gross IS NOT NULL
) RankedMovies
WHERE Rank = 1 AND GROSS != 0
ORDER BY GROSS DESC; 
```

### 1️⃣4️⃣ Create a new column that categorizes movies as **"Hit" or "Flop"** based on Gross revenue (above/below average).
```sql
-- Write your SQL answer here
SELECT 
    movie_name, 
    gross_revenue,
    CASE 
        WHEN gross_revenue > (SELECT AVG(gross_revenue) FROM movies) THEN 'Hit'
        ELSE 'Flop'
    END AS movie_category
FROM 
    movies;
```

### 1️⃣5️⃣ Find the **actor who has appeared in the most movies with an IMDB rating above 8.0**.
```sql
-- Write your SQL answer here

