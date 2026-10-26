# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/Swati-Gupta92/Netflix-SQL-Analysis/blob/main/logo.png?raw=true)
#  Dashboard using Power BI

![](https://github.com/Swati-Gupta92/Netflix-SQL-Analysis/blob/main/netflix_teblue.jpg?raw=true)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

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
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT * 
FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie

```sql
SELECT 
    *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

**Objective:** Find the movie with the longest duration.

### 6. Find Content Added During the Last Available Year

```sql
SELECT *
FROM public.netflix
WHERE TO_DATE(TRIM(date_added), 'FMMonth DD, YYYY')
      >= DATE '2020-09-26'
  AND TO_DATE(TRIM(date_added), 'FMMonth DD, YYYY')
      <= DATE '2021-09-25';
```

**Objective:** Retrieve Netflix titles added during the last available year covered by the dataset.

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

**Objective:** List all content directed by 'Rajiv Chilaka'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

**Objective:** Calculate and rank years by the average number of content releases by India.

### 11. List All Movies that are Documentaries

```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find All Content Without a Director

```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```
**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

### 16.⁠ ⁠Count the Number of Movies and TV Shows

```sql
SELECT
    type,
    COUNT(*) AS total_titles
FROM public.netflix
GROUP BY type
ORDER BY total_titles DESC;
```

**Objective:** Compare the total number of Movies and TV Shows in the dataset.

### 17.⁠ ⁠Analyze Netflix Titles by Release Year

```sql
SELECT
    release_year,
    COUNT(*) AS total_titles
FROM public.netflix
GROUP BY release_year
ORDER BY release_year DESC;
```

**Objective:** Analyze the number of titles associated with each release year and explore content release trends.

### 18.⁠ ⁠Analyze Content Rating Distribution

```sql
SELECT
    rating,
    COUNT(*) AS total_titles
FROM public.netflix
GROUP BY rating
ORDER BY total_titles DESC;
```

**Objective:** Identify the most common content ratings across the Netflix dataset.

### 19.⁠ ⁠Identify the Top 10 Directors by Number of Titles

```sql
SELECT
    director,
    COUNT(*) AS total_titles
FROM public.netflix
WHERE director IS NOT NULL
GROUP BY director
ORDER BY total_titles DESC
LIMIT 10;
```

**Objective:** Identify the ten directors associated with the highest number of titles in the dataset.

### 20.Find the Top 5 Countries by Number of Movies

```sql
SELECT country, COUNT(*) AS total_movies
FROM public.netflix
WHERE type = 'Movie'
  AND country IS NOT NULL
GROUP BY country
ORDER BY total_movies DESC
LIMIT 5;
```

**Objective:** Identify the top five countries with the highest number of movies on Netflix.


## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.
- **Additional Analysis:** Four additional SQL queries were developed and tested using PostgreSQL.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.



## Author - Swati Gupta

Developed as a portfolio project to strengthen my practical SQL skills and explore real-world data analysis using PostgreSQL.

## Project Highlights:

- Analyzed 8,807 Netflix titles.
- Compared Movies and TV Shows.
- Explored content ratings and release-year trends.


## ⁠LinkedIn: [Connect with me professionally](https://www.linkedin.com/in/swati-gupta-15798233b/)
