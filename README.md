# Netflix Movie and TV Shoes Data Analysis using SQL

![Netflix logo](https://github.com/Princesaini25/netflix_sql_project/blob/main/logo.png)

## 📌 Project Overview

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

---

## 🎯 Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

---

CREATE TABLE netflix (
    show_id      VARCHAR(10),
    type         VARCHAR(10),
    title        VARCHAR(150),
    director     VARCHAR(250),
    casts        VARCHAR(1000),
    country      VARCHAR(150),
    date_added   VARCHAR(50),
    release_year INT,
    rating       VARCHAR(10),
    duration     VARCHAR(15),
    listed_in    VARCHAR(150),
    description  VARCHAR(300)
);


SELECT*FROM netflix;

--1.Count the number of Movies vs TV Shows--

SELECT type, COUNT(*) AS total_content
FROM netflix
GROUP BY type;

--2.Find the most common rating for Movies and TV Shows--

SELECT type, rating
FROM (
    SELECT type, rating,
           COUNT(*) AS rating_count,
           RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS rnk
    FROM netflix
    GROUP BY type, rating
) AS t
WHERE rnk = 1;

--3.List all Movies released in a specific year (e.g., 2020)--

SELECT title, release_year
FROM netflix
WHERE type = 'Movie'
  AND release_year = 2020;

--4.Find the top 5 countries with the most content--

  SELECT TRIM(c) AS country, COUNT(*) AS total_content
FROM netflix,
     UNNEST(string_to_array(country, ',')) AS c
WHERE country IS NOT NULL
GROUP BY TRIM(c)
ORDER BY total_content DESC
LIMIT 5;

--5.Identify the longest movie on tha basis of duration --

SELECT title, duration
FROM netflix
WHERE type = 'Movie'
  AND duration IS NOT NULL
ORDER BY CAST(REPLACE(duration, ' min', '') AS INT) DESC
LIMIT 1;

--6.Find content added in the last 5 years--

SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';

--7.Find all content directed by a specific director (e.g., 'Rajiv Chilaka')--

SELECT title, type
FROM netflix
WHERE director LIKE '%Rajiv Chilaka%';

--8.List all TV Shows with more than 5 seasons--

SELECT title, duration
FROM netflix
WHERE type = 'TV Show'
  AND CAST(SPLIT_PART(duration, ' ', 1) AS INT) > 5;

--9. Count the number of content items in each genre--

SELECT TRIM(genre) AS genre, COUNT(*) AS total_content
FROM netflix,
     UNNEST(string_to_array(listed_in, ',')) AS genre
GROUP BY TRIM(genre)
ORDER BY total_content DESC;

--10.Find each year and the average content release by India. top 5 years--

SELECT release_year,
       COUNT(*) AS yearly_content,
       ROUND(
         COUNT(*)::numeric /
         (SELECT COUNT(*) FROM netflix WHERE country LIKE '%India%') * 100, 2
       ) AS avg_content_per_year
FROM netflix
WHERE country LIKE '%India%'
GROUP BY release_year
ORDER BY avg_content_per_year DESC
LIMIT 5;

--11.List all Movies that are Documentaries--

SELECT title
FROM netflix
WHERE type = 'Movie'
  AND listed_in LIKE '%Documentaries%';

--12.Find all content without a director--

SELECT COUNT(*) AS missing_director_count
FROM netflix
WHERE director IS NULL OR director = '';

--13.Find how many movies a specific actor (e.g., 'Salman Khan') appeared in over the last 10 years--

SELECT COUNT(*) AS movie_count
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;

--14.Find the top 10 actors who appeared in the highest number of Movies produced in India--

SELECT TRIM(actor) AS actor, COUNT(*) AS total_movies
FROM netflix,
     UNNEST(string_to_array(casts, ',')) AS actor
WHERE country LIKE '%India%'
  AND type = 'Movie'
GROUP BY TRIM(actor)
ORDER BY total_movies DESC
LIMIT 10;

--15.Categorize content as 'Good' or 'Bad' based on keywords in description--

SELECT category, COUNT(*) AS total_content
FROM (
    SELECT
        CASE
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%'
                THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized
GROUP BY category;

--16.Find the year-wise growth of Netflix content additions--

SELECT EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) AS year_added,
       COUNT(*) AS content_added
FROM netflix
WHERE date_added IS NOT NULL
GROUP BY year_added
ORDER BY year_added;

--17.Find all the seasons available for each TV Show--

SELECT title,
       CAST(SPLIT_PART(duration, ' ', 1) AS INT) AS total_seasons
FROM netflix
WHERE type = 'TV Show'
ORDER BY total_seasons DESC;

--18.Find content that does not have a country listed--

SELECT show_id, title, type
FROM netflix
WHERE country IS NULL OR country = '';

--19.Find how many movies vs TV shows were added each year (Year-over-Year comparison)--

SELECT EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) AS year_added,
       type,
       COUNT(*) AS total_content
FROM netflix
WHERE date_added IS NOT NULL
GROUP BY year_added, type
ORDER BY year_added, type;

--20.Find the top 5 most common genres for each country (e.g., India, United States)--

SELECT country, genre, total_content
FROM (
    SELECT TRIM(country) AS country,
           TRIM(genre) AS genre,
           COUNT(*) AS total_content,
           RANK() OVER (PARTITION BY TRIM(country) ORDER BY COUNT(*) DESC) AS rnk
    FROM netflix,
         UNNEST(string_to_array(listed_in, ',')) AS genre
    WHERE country IN ('India', 'United States')
    GROUP BY TRIM(country), TRIM(genre)
) AS ranked
WHERE rnk <= 5;

## 📊 Findings

1. **Content Distribution** — Netflix has significantly more Movies (6,131) than TV Shows (2,676), making up ~69.6% of total content.
2. **Common Ratings** — `TV-MA` is the most frequent rating for TV Shows, while `TV-14` dominates Movies — indicating Netflix primarily targets mature audiences.
3. **Top Countries** — The United States leads with 3,690 titles, followed by India (1,046), UK (806), Japan (717), and South Korea (592).
4. **India Content Trend** — India's content additions peaked between 2018–2020, reflecting Netflix's aggressive push into the Indian market.
5. **Genre Popularity** — International Movies, Dramas, and Comedies are consistently the top genres across the platform.
6. **Data Quality** — 2,634 records (~30%) have missing director information and 831 records have no country listed.
7. **Content Growth** — Netflix showed rapid content growth from 2016 to 2019, after which the pace slightly slowed.
8. **Long TV Shows** — Very few TV Shows exceed 5 seasons, suggesting Netflix favors limited series formats.

---

## 🏁 Conclusions

This analysis reveals several key patterns in Netflix's content strategy:

- Netflix is movie-heavy but is increasingly investing in original TV Shows.
- The platform heavily targets mature audiences based on rating distribution.
- USA and India are the dominant content markets — making them priority regions for localization.
- Dramas and International content consistently outperform other genres, highlighting a preference for story-driven global content.
- The presence of significant missing data (director, country fields) indicates a need for better data governance and enrichment.

This project demonstrates how SQL can uncover actionable business insights from raw streaming data — directly applicable to roles in Data Analysis, Business Intelligence, and Product Analytics.
