# CODE_NETFLIX_SQL
![Netflix](https://github.com/ishikabhaorjar-dotcom/Code_netflix_SQL/blob/main/images.jpg)

## Ishika's Discover


Project Overview:
This project focuses on analyzing Netflix dataset using Pure SQL. The objective of the project is to explore and understand the data through different analytical queries. The dataset contains information about Netflix movies and TV shows such as title, type, director, cast, country, release year, rating, and duration.

In this project, 15 different analytical questions are solved using SQL queries. These questions help in extracting meaningful insights from the dataset such as identifying the number of movies and TV shows, analyzing content distribution by country, finding the most common ratings, examining release trends over the years, and many more.

The project demonstrates the practical use of SQL concepts including:
- SELECT statements
- Filtering with WHERE clause
- Aggregation functions (COUNT, MAX, MIN, AVG)
- GROUP BY and ORDER BY
- Data filtering and sorting
- Basic data exploration techniques

This project helps in strengthening SQL querying skills and shows how SQL can be used for real-world data analysis.

Tools Used:
- MySQL
- Netflix Dataset (CSV)


SCHEMAS 


### Netflix Data Analysis using SQL

Instructions to Run the Project:
1. First, create a table named 'netflix' using the schema provided below.
2. After creating the table, refresh the database schema.
3. Import the Netflix dataset (CSV file) into the 'netflix' table.
4. While importing the CSV file, make sure the "Header" option is enabled so that the column names are correctly recognized.
5. Save the import settings and load the data into the table.
6. Once the dataset is successfully imported, run the SQL queries below to perform the analysis.

This project contains 15 business questions solved using SQL queries to analyze Netflix content.


-- Remove the table if it already exists
DROP TABLE IF EXISTS netflix;

-- Create Netflix Table
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

-- View the dataset
SELECT * FROM netflix;

-- Check the different types of content available
SELECT DISTINCT type
FROM netflix;

-- =====================================================
-- 15 Business Problems
-- =====================================================

-- 1. Count the number of Movies vs TV Shows
SELECT 
    type,
    COUNT(*) AS total_content
FROM netflix
GROUP BY type;


-- 2. Find the most common rating for Movies and TV Shows
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


-- 3. List all movies released in a specific year (Example: 2020)
SELECT * 
FROM netflix
WHERE release_year = 2020;


-- 4. Find the Top 5 Countries with the most content on Netflix
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


-- 5. Identify the longest movie based on duration
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;


-- 6. Find content added in the last 5 years
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';


-- 7. Find all movies/TV shows directed by 'Rajiv Chilaka'
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';


-- 8. List all TV shows with more than 5 seasons
SELECT *
FROM netflix
WHERE type = 'TV Show'
AND SPLIT_PART(duration, ' ', 1)::INT > 5;


-- 9. Count the number of content items in each genre
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY genre;


-- 10. Find each year and the average number of content releases in India
-- Return the Top 5 years with the highest average release percentage

SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release_percentage
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release_percentage DESC
LIMIT 5;


-- 11. List all movies that are Documentaries
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';


-- 12. Find all content without a director
SELECT * 
FROM netflix
WHERE director IS NULL;


-- 13. Find how many movies actor 'Salman Khan' appeared in during the last 10 years
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;


-- 14. Find the Top 10 actors who appeared in the highest number of movies produced in India
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*) AS total_movies
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY total_movies DESC
LIMIT 10;


-- 15. Categorize content based on the presence of 'Kill' or 'Violence' keywords in the description
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
  
