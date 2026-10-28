
# 🎬 Netflix Movies and TV Shows Data Analysis Using SQL

## Overview

This project focuses on analyzing Netflix movies and TV shows data using SQL and PostgreSQL. The objective is to explore Netflix's content library, identify trends, and answer 15 business questions using SQL queries.

Through this project, I practiced data exploration, aggregation, string manipulation, Common Table Expressions (CTEs), window functions, and subqueries to extract meaningful insights from the dataset.

## Objectives

- Analyze the distribution of movies and TV shows on Netflix.
- Identify the most common content ratings.
- Explore content by release year, country, genre, and duration.
- Analyze Indian content and identify popular actors.
- Identify content without director information.
- Explore documentary movies and TV shows with multiple seasons.
- Categorize content descriptions using specific keywords.
- Apply SQL concepts to solve real-world business questions.


## Business Problems and Solutions

### 1. Count the Number of Movies vs. TV Shows

**Objective:** Understand the distribution of content types available on Netflix.

```sql
SELECT
    type,
    COUNT(*) AS total_content
FROM netflix
GROUP BY type;
```

### 2. Find the Most Common Rating for Movies and TV Shows

**Objective:** Identify the most frequently occurring content rating for each type.

```sql
WITH RatingCounts AS (
    SELECT
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT
        type,
        rating,
        rating_count,
        RANK() OVER (
            PARTITION BY type
            ORDER BY rating_count DESC
        ) AS rating_rank
    FROM RatingCounts
)
SELECT
    type,
    rating AS most_frequent_rating,
    rating_count
FROM RankedRatings
WHERE rating_rank = 1;
```

### 3. Find Content Released in 2020

**Objective:** Retrieve titles released in 2020. To retrieve movies only, use the additional `type = 'Movie'` filter.

```sql
SELECT *
FROM netflix
WHERE release_year = 2020;
```

### 4. Find the Top 5 Countries with the Most Netflix Content

**Objective:** Identify the countries associated with the highest number of titles.

```sql
SELECT
    TRIM(country_name) AS country,
    COUNT(*) AS total_content
FROM netflix
CROSS JOIN LATERAL
    UNNEST(STRING_TO_ARRAY(country, ',')) AS c(country_name)
WHERE country IS NOT NULL
GROUP BY TRIM(country_name)
ORDER BY total_content DESC
LIMIT 5;
```

### 5. Find the Longest Movie

**Objective:** Identify the movie with the greatest duration.

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
  AND duration ~ '^[0-9]+ min$'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
LIMIT 1;
```

### 6. Find Content Added in the Last 5 Years

**Objective:** Retrieve titles added to Netflix during the last five years.

```sql
SELECT *
FROM netflix
WHERE date_added IS NOT NULL
  AND TRIM(date_added) <> ''
  AND TO_DATE(TRIM(date_added), 'Month DD, YYYY')
      >= CURRENT_DATE - INTERVAL '5 years';
```

### 7. Find Titles Directed by Rajiv Chilaka

**Objective:** Retrieve titles associated with director Rajiv Chilaka.

```sql
SELECT DISTINCT n.*
FROM netflix AS n
CROSS JOIN LATERAL
    UNNEST(STRING_TO_ARRAY(n.director, ',')) AS d(director_name)
WHERE TRIM(d.director_name) = 'Rajiv Chilaka';
```

### 8. Find TV Shows with More Than 5 Seasons

**Objective:** Identify TV shows with more than five seasons.

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND duration ~ '^[0-9]+ Seasons?$'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

### 9. Count Content Items by Genre

**Objective:** Analyze how Netflix titles are distributed across genres.

```sql
SELECT
    TRIM(genre_name) AS genre,
    COUNT(*) AS total_content
FROM netflix
CROSS JOIN LATERAL
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS g(genre_name)
WHERE listed_in IS NOT NULL
GROUP BY TRIM(genre_name)
ORDER BY total_content DESC;
```

### 10. Find the Top 5 Release Years for Indian Content

**Objective:** Identify the five release years with the highest number of Indian titles and calculate each year's share of Indian content.

```sql
WITH IndianContent AS (
    SELECT DISTINCT
        n.show_id,
        n.release_year
    FROM netflix AS n
    CROSS JOIN LATERAL
        UNNEST(STRING_TO_ARRAY(n.country, ',')) AS c(country_name)
    WHERE TRIM(c.country_name) = 'India'
),
YearlyReleases AS (
    SELECT
        release_year,
        COUNT(*) AS total_release
    FROM IndianContent
    GROUP BY release_year
),
TotalIndianContent AS (
    SELECT COUNT(*) AS total_content
    FROM IndianContent
)
SELECT
    yr.release_year,
    yr.total_release,
    ROUND(
        yr.total_release::NUMERIC
        / NULLIF(tic.total_content, 0) * 100,
        2
    ) AS percentage_of_indian_content
FROM YearlyReleases AS yr
CROSS JOIN TotalIndianContent AS tic
ORDER BY yr.total_release DESC
LIMIT 5;
```

### 11. Find Documentary Content

**Objective:** Retrieve titles categorized under documentaries.

```sql
SELECT *
FROM netflix
WHERE listed_in ILIKE '%Documentaries%';
```

### 12. Find Content Without a Director

**Objective:** Identify titles with missing director information.

```sql
SELECT *
FROM netflix
WHERE director IS NULL
   OR TRIM(director) = '';
```

### 13. Find Movies Featuring Salman Khan Released in the Last 10 Years

**Objective:** Retrieve movies featuring Salman Khan released within the last ten calendar years.

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
  AND casts ILIKE '%Salman Khan%'
  AND release_year >= EXTRACT(YEAR FROM CURRENT_DATE)::INT - 9;
```

### 14. Find the Top 10 Actors in Indian Content

**Objective:** Identify actors appearing in the highest number of distinct Indian titles.

```sql
WITH IndianTitles AS (
    SELECT DISTINCT n.show_id, n.casts
    FROM netflix AS n
    CROSS JOIN LATERAL
        UNNEST(STRING_TO_ARRAY(n.country, ',')) AS c(country_name)
    WHERE TRIM(c.country_name) = 'India'
),
IndianActors AS (
    SELECT
        TRIM(actor_name) AS actor,
        show_id
    FROM IndianTitles
    CROSS JOIN LATERAL
        UNNEST(STRING_TO_ARRAY(casts, ',')) AS a(actor_name)
    WHERE casts IS NOT NULL
)
SELECT
    actor,
    COUNT(DISTINCT show_id) AS total_titles
FROM IndianActors
WHERE actor <> ''
GROUP BY actor
ORDER BY total_titles DESC
LIMIT 10;
```

### 15. Categorize Content Using Description Keywords

**Objective:** Classify titles as `Bad` when their descriptions contain the words "kill" or "violence", and `Good` otherwise. This is a simple keyword-based SQL exercise, not a reliable assessment of content suitability.

```sql
SELECT
    CASE
        WHEN description ILIKE '%kill%'
          OR description ILIKE '%violence%'
        THEN 'Bad'
        ELSE 'Good'
    END AS category,
    type,
    COUNT(*) AS content_count
FROM netflix
GROUP BY category, type
ORDER BY category, type;
```

## Author

**Aravind K**  
Data Analyst | MIS Reporting | SQL | Power BI | Python  
Bengaluru, India

Thank you for checking out my project! Feedback and suggestions are welcome.
