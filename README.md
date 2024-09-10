# Netflix Movies and TV Shows Data Analysis using SQL

## Overview
This project conducts an in-depth analysis of Netflix's movies and TV shows data using SQL. The primary objective is to derive valuable insights and answer business-related queries based on the dataset. The project involves exploring content distribution, ratings, release years, countries, and durations, along with other insights.

## Objectives
- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on keywords.

## Dataset
The data for this analysis is sourced from Netflix’s public dataset available on Kaggle. It contains information on movies and TV shows, including title, director, cast, country, release year, duration, genre, and more.

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
GROUP BY type 
ORDER BY type;
```
**Objective**: Determine the distribution of content types on Netflix.

---

### 2. Find the Most Common Rating for Movies and TV Shows
```sql
WITH cte AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS common_rating,
        ROW_NUMBER() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) AS rn 
    FROM netflix
    GROUP BY type, rating
)
SELECT type, rating, common_rating 
FROM cte 
WHERE rn = 1;
```
**Objective**: Identify the most frequently occurring rating for each type of content.

---

### 3. List All Movies Released in a Specific Year (e.g., 2020)
```sql
SELECT * 
FROM netflix 
WHERE type = 'Movie' 
  AND release_year = 2020;
```
**Objective**: Retrieve all movies released in the year 2020.

---

### 4. Find the Top 5 Countries with the Most Content on Netflix
```sql
SELECT *
FROM (
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS Country,
        COUNT(*) AS country_count
    FROM netflix
    GROUP BY country
) AS t
WHERE Country IS NOT NULL 
ORDER BY country_count DESC
LIMIT 5;
```
**Objective**: Identify the top 5 countries with the most content on Netflix.

---

### 5. Identify the Longest Movie
```sql
SELECT * 
FROM netflix
WHERE type = 'Movie' 
ORDER BY CAST(SPLIT_PART(duration, ' ', 1) AS INT) DESC;
```
**Objective**: Find the movie with the longest duration.

---

### 6. Find Content Added in the Last 5 Years
```sql
SELECT release_year, COUNT(*)
FROM netflix
GROUP BY release_year
ORDER BY release_year DESC 
LIMIT 5;
```
**Objective**: Retrieve content added to Netflix in the last 5 years.

---

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'
```sql
SELECT *
FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%';
```
**Objective**: List all content directed by 'Rajiv Chilaka'.

---

### 8. List All TV Shows with More Than 5 Seasons
```sql
SELECT title, duration 
FROM netflix
WHERE type = 'TV Show' 
  AND CAST(SPLIT_PART(duration, ' ', 1) AS INT) > 5 
ORDER BY CAST(SPLIT_PART(duration, ' ', 1) AS INT) DESC;
```
**Objective**: Identify TV shows with more than 5 seasons.

---

### 9. Count the Number of Content Items in Each Genre
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS Listed_in, 
    COUNT(*) 
FROM netflix
GROUP BY Listed_in
ORDER BY COUNT(*) DESC;
```
**Objective**: Count the number of content items in each genre.

---

### 11. List All Movies that are Documentaries
```sql
SELECT * 
FROM netflix
WHERE listed_in ILIKE '%Documentaries%';
```
**Objective**: Retrieve all movies classified as documentaries.

---

### 12. Find All Content Without a Director
```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```
**Objective**: List all content that does not have a director.

---

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years
```sql
SELECT * 
FROM netflix
WHERE casts ILIKE '%Salman Khan%' 
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```
**Objective**: Count the number of movies featuring 'Salman Khan' in the last 10 years.

---

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India
```sql
WITH cte AS (
    SELECT
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
        *
    FROM netflix
)
SELECT actor, COUNT(*) 
FROM cte 
WHERE country = 'India'
GROUP BY actor 
ORDER BY COUNT(*) DESC 
LIMIT 10;
```
**Objective**: Identify the top 10 actors with the most appearances in Indian-produced movies.

---

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords
```sql
SELECT 
    CASE 
        WHEN description ILIKE '%Kill%' OR description ILIKE '%Violence%' 
        THEN 'Bad' 
        ELSE 'Good' 
    END AS Category,
    COUNT(*) AS content 
FROM netflix  
GROUP BY Category
ORDER BY content DESC;
```
**Objective**: Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise.

---

## Findings and Conclusion
- **Content Distribution**: Netflix offers a diverse range of content, with both movies and TV shows widely represented.
- **Most Common Ratings**: Ratings insights help in understanding the content's target audience and popularity.
- **Geographical Insights**: The top countries and content release trends provide a regional perspective.
- **Content Categorization**: The analysis of keywords such as 'kill' and 'violence' assists in categorizing and understanding the nature of Netflix's content.
