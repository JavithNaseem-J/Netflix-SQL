# Netflix Movies and TV Shows Data Analysis using SQL

## Overview
This project conducts an in-depth analysis of Netflix's movies and TV shows data using SQL. The primary objective is to derive valuable insights and answer business-related queries based on the dataset. The project involves exploring content distribution, ratings, release years, countries, and durations, along with other insights.

## Objectives
- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on keywords.

## Dataset
The data for this analysis is sourced from Netflix’s public dataset available on [Kaggle](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download).

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
![image](https://github.com/user-attachments/assets/f6d260b4-7b15-4814-ade3-bd47646ec368)

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
![image](https://github.com/user-attachments/assets/2c647974-bdbe-44aa-94c3-bab047073f38)

---

### 3. List All Movies Released in a Specific Year (e.g., 2020)
```sql
SELECT * 
FROM netflix 
WHERE type = 'Movie' 
  AND release_year = 2020;
```
![image](https://github.com/user-attachments/assets/68e89c07-867f-426e-8e19-c516ec9fd2ba)

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
![image](https://github.com/user-attachments/assets/bf0cd7f2-682d-4412-9009-4380c09acd60)

---

### 5. Identify top 10 Longest Movie
```sql
with cte as (SELECT 
		Title,release_year,rating,duration,row_number() over(order by CAST(split_part(duration, ' ', 1) AS int ) DESC) AS RN
FROM 
	netflix
WHERE 
	type = 'Movie' and  duration is not null)
SELECT
	* 
FROM 
	CTE 
WHERE 
	RN<=10;
```
![image](https://github.com/user-attachments/assets/1c058159-2d12-4629-857f-caf0d0393c31)

---

### 6. Find Content Added in the Last 5 Years
```sql
SELECT release_year, COUNT(*)
FROM netflix
GROUP BY release_year
ORDER BY release_year DESC 
LIMIT 5;
```
![image](https://github.com/user-attachments/assets/b4f6a0b3-9e48-40f7-98af-947e3d67e55f)

---

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'
```sql
select 
	TITLE,DIRECTOR,RELEASE_YEAR
From
	netflix
where 
	director ilike '%Rajiv Chilaka%'
ORDER BY
	RELEASE_YEAR ASC;
```
![image](https://github.com/user-attachments/assets/49c2a1fb-4d84-442e-ab5a-dac13dca05cb)

---

### 8. List All TV Shows with More Than 5 Seasons
```sql
select 
	title,duration 
from 
	netflix
WHERE 
	type='TV Show' and CAST(split_part(duration, ' ', 1) AS int) >= 10 
ORDER BY 
	2 DESC;
```
![image](https://github.com/user-attachments/assets/a8db08d9-94ef-4324-98b8-423c765a0a9e)

---

### 9. Count the Number of Content Items in Each Genre
```sql
WITH CTE AS (select 
	trim(unnest(string_to_array(listed_in,','))) as Listed_in
from 
	netflix)
SELECT
	listed_in,count(*)
FROM
	CTE
GROUP BY 
	1
ORDER BY
	2 DESC;
```
![image](https://github.com/user-attachments/assets/49e858f9-1086-4108-842e-a319c7ccafb1)

---

### 10.Find each year and the average numbers of content release in India on netflix.
```sql
SELECT 
	COUNTRY,RELEASE_YEAR,COUNT(*) AS NO_OF_CONTENT,
	ROUND(CAST(COUNT(*)AS NUMERIC)/(SELECT COUNT(*) FROM NETFLIX WHERE COUNTRY='India')*100,2)
FROM 
	NETFLIX
WHERE 
	COUNTRY='India'
GROUP BY 
	1,2
ORDER BY 
	3 DESC LIMIT 5;
```
![image](https://github.com/user-attachments/assets/462e0ec6-efb7-4b3f-a149-15fc4cf55844)


### 11. List All Movies that are Documentaries
```sql
SELECT * 
FROM netflix
WHERE listed_in ILIKE '%Documentaries%';
```
![image](https://github.com/user-attachments/assets/5d743f0b-0d33-4ad5-b24d-5e28357b3ca0)

850+ movies classified as documentaries.
---

### 12. Find All Content Without a Director
```sql
select
	TYPE,SUM(CASE WHEN DIRECTOR IS NULL THEN 1 ELSE 0 END) AS WITHOUT_DIRECTOR
from
	netflix
GROUP BY
	1;
```
![image](https://github.com/user-attachments/assets/e30fa688-7844-4b59-af23-7df7e012940d)

---

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years
```sql
SELECT * 
FROM netflix
WHERE casts ILIKE '%Salman Khan%' 
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```
![image](https://github.com/user-attachments/assets/0cd62739-0db9-45ca-9f7d-e73c067d4669)

---

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India
```sql
WITH cte AS (
    SELECT
        unnest(string_to_array(country, ',')) AS countrys,
        unnest(string_to_array(casts, ',')) AS actor,
        *
    FROM
        netflix
)
SELECT 
    actor, COUNT(*) 
FROM 
    cte 
WHERE 
    countrys = 'India' AND actor is not null
GROUP BY 
    actor 
ORDER BY 
    COUNT(*) DESC 
LIMIT 10;
```
![image](https://github.com/user-attachments/assets/a9eaf5cd-7434-4aa4-b331-f80cfab83190)

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
![image](https://github.com/user-attachments/assets/63cdf372-2cd9-43e3-b009-79e395d191e5)

---

## Findings and Conclusion
- **Content Distribution**: Netflix offers a diverse range of content, with both movies and TV shows widely represented.
- **Most Common Ratings**: Ratings insights help in understanding the content's target audience and popularity.
- **Geographical Insights**: The top countries and content release trends provide a regional perspective.
- **Content Categorization**: The analysis of keywords such as 'kill' and 'violence' assists in categorizing and understanding the nature of Netflix's content.
