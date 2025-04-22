# Netflix Movies and TV Shows Data Analysis using SQL

![Netflix Logo](https://github.com/prayag-dave-01/sql_p3_netflix/blob/main/logo.png?raw=true)

## Project Overview

**Project Title**: Netflix Movies and TV Shows Data Analysis using SQL
**Level**: Intermediate to Advanced
**Database**: `p3_netflix_db`

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `p3_netflix_db`.
- **Table Creation**: A table named `netflix` is created to store the sales data. The table structure includes columns for show_id, type, title, director, cast, country, date_added, release_year, rating, duration, listed_in, and description

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

### 2. Data Exploration & Cleaning
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:
- `type`: Show type whether movie or tv show
- `title`: The name of the movie/tv show.
- `listed_in`: Type of the genre
- `release_year`: The year in which movie/tv show was released.
- `date_added`: The day when content was added to Netflix.
- Various metrics such as `cast`, `description`, `duration`, `rating`, and more.

```sql
SELECT * FROM netflix;

SELECT 
    COUNT(*) as total_content
FROM netflix;

SELECT 
    DISTINCT type
FROM netflix;

SELECT MAX(release_year)
FROM netflix;

SELECT MIN(release_year)
FROM netflix;

SELECT MAX(date_added)
FROM netflix;

SELECT MIN(date_added)
FROM netflix;
```

### 3. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data. Based on the business problems, queries were written as mentioned below to achieve the outcome.

#### Easy Queries
- Simple data retrieval, filtering, and basic aggregations.
  
#### Medium Queries
- More complex queries involving grouping, and aggregation functions.
  
#### Advanced Queries
- Nested subqueries, window functions, CTEs, and performance optimization.
  
### 4. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT 
     type,
	 COUNT (*) as total_content
FROM netflix
GROUP BY 1;
```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the most common rating for movies and TV shows

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
	  RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) as rank
  FROM netflix
  GROUP BY 1, 2
) as t1
WHERE rank = 1;
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List all movies released in a specific year (e.g., 2020)

```sql
SELECT
    title
FROM netflix
WHERE
    type = 'Movie'
    AND
    release_year = 2020;
```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the top 5 countries with the most content on Netflix

```sql
SELECT
    UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
	COUNT(show_id) as total_content
FROM netflix
WHERE country IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the longest movie

```sql
SELECT
    title
FROM netflix
WHERE 
   type = 'Movie'
   AND
   duration = (SELECT MAX(duration) FROM netflix);
```

**Objective:** Find the movie with the longest duration.

### 6. Find content added in the last 5 years

```sql
SELECT 
     *
FROM netflix
WHERE 
   TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find all the movies/TV shows by director 'Rajiv Chilaka'

```sql
SELECT
    title,
	type
FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%';

---Alternative

SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

**Objective:** List all content directed by 'Rajiv Chilaka'. UNNEST can be used since few movies/TV shows have multiple directors.

### 8. List all TV shows with more than 5 seasons

```sql
SELECT
    title
FROM netflix
WHERE 
    type = 'TV Show'
    AND
	SPLIT_PART(duration, ' ', 1)::numeric > 5;
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the number of content items in each genre

```sql
SELECT
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(show_id) as total_content
FROM netflix
GROUP BY 1;
```

**Objective:** Count the number of content items in each genre. UNNEST is used since few movies/TV shows have multiple genres.

### 10.Find each year and the average numbers of content release in India on netflix. Return top 5 year with highest avg content release!

```sql
SELECT 
    EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD YYYY')) as year,
	COUNT(*) as yearly_content,
	ROUND(
	COUNT (*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country = 'India') * 100, 2) as avg_content
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 3 DESC
LIMIT 5;
```

**Objective:** Calculate and rank years by the average number of content releases by India.

### 11. List all movies that are documentaries

```sql
SELECT
    title
FROM netflix
WHERE listed_in ILIKE '%documentaries%';
```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find all content without a director

```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

**Objective:** List content that does not have a director.

### 13. Fetch all movies where actor 'Salman Khan' appeared in last 10 years!

```sql
SELECT 
    type,
	title,
	casts,
	release_year
FROM netflix
WHERE 
    casts ILIKE '%Salman Khan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
	COUNT(*) total_movies
FROM netflix
WHERE country ILIKE '%India%'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category. Write 2 queries (one usineg CTE and other using subquery)

```sql
-- CTE
WITH new_table
AS
(
 SELECT
      *,
	  CASE 
	  WHEN
	     description ILIKE '%kill%' OR
	 	description ILIKE '%violence%' THEN 'Bad'
	  ELSE 'Good'
	  END category
 FROM netflix
)
 SELECT
    category,
	COUNT(*) as total_movies
FROM new_table
GROUP BY 1;

-- SUBQUERY
SELECT
    category,
	COUNT(*) as total_movies
FROM
(
 SELECT
      *,
	  CASE 
	  WHEN
	     description ILIKE '%kill%' OR
	 	description ILIKE '%violence%' THEN 'Bad'
	  ELSE 'Good'
	  END category
 FROM netflix
)
GROUP BY 1;
```

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

## Author - Prayag Dave

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!

- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/prayag-dave-56b3681b3)
- **Mail**: prayagdavework@gmail.com

Thank you for your support, and I look forward to connecting with you!
