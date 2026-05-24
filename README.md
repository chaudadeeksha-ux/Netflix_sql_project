# Netflix_sql_project

<img width="2226" height="678" alt="logo" src="https://github.com/user-attachments/assets/9661c782-3ae3-4156-9409-18f0d7d2d1c4" />

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Schema

```sql
drop table if exists netflix;
create table netflix(
   show_id	varchar(5) primary key,
   type	varchar(10),
    title	varchar(150),
    director	varchar(550),
    casts	varchar(10000),
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
select type, count(*) 
from netflix
group by type;
```

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
select 
   type , 
   rating
from (select 
      type, 
	  rating, 
	  count(*),
	  rank() over(partition by type  order by count(*) desc ) as ranking
from netflix
group by 1,2) as t
where ranking = 1;
```


### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
Select * from netflix
WHERE type = 'Movie'
 and release_year = 2020;
```

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
select 
UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
     count(show_id) as total_content
	 from netflix
	 group by 1;
```

### 5. Identify the Longest Movie	 

```sql
SELECT *,
      type , 
       duration,
       CAST(REGEXP_REPLACE(duration, '[^0-9]', '','g') AS INT) as num
FROM netflix
where type = 'Movie'
order by num desc
limit 1 offset 3;
```

### 6. Find Content Added in the Last 5 Years

```sql
select *
from netflix
where To_date(date_added , 'month dd , yyyy') >= current_date - interval '5 years'
```

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql

select 
   *
from 
(
select *,
UNNEST(STRING_TO_ARRAY(director, ',')) AS uni_directors
from netflix
) as t
where uni_directors = 'Rajiv Chilaka';
```

```sql
select * from netflix
where director like '%Rajiv Chilaka%';
```

### 8. List All TV Shows with More Than 5 Seasons

```sql
select 
      type ,
	  duration,
      split_part(duration , ' ', 1):: numeric as season
	  from netflix
where type = 'TV Show' and 
split_part(duration , ' ', 1):: numeric > 5
order by season desc;
```

### 9. Count the Number of Content Items in Each Genre

```sql
select 
     unnest(string_to_array(listed_in, ','))  as genre,
	 count(show_id) as total_content
from netflix
group by genre;
```

### 10. Find each year and the average numbers of content release in India on netflix. return top 5 year with highest avg content release!

```sql
select 
   extract(year from To_date(date_added, 'Month dd , yyyy')) as year,
   count(*),
   count(*)::numeric / (select count(*) from netflix where country = 'india')::numeric * 100 as avg_content_year
  from netflix
where country =  'India'
group by 1;
```

### 11. List All Movies that are Documentaries

```sql
select  * 
from netflix
where  
listed_in Ilike '%Documentaries%';
```

### 12. Find All Content Without a Director

```sql
select * 
from netflix
where director Is Null
```


### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
select * from 
netflix
where 
     casts like '%Salman Khan%'
and 
release_year >  extract(year from current_date) - 10;
```

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
select 
    unnest(string_to_array(casts , ',')) as actor,
	count(*) as total_content
from netflix
where country ilike '%india%'
group by 1
order by 2 desc
limit 10;
```

### 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.

```sql
with new_table
as 
(
select 
     * , 
	 case
	 when 
	    description ilike '%kill%' or 
		description ilike '%violence%' then 'bad_content'
    else 'good content'
	end category
	from netflix
)	
select 
category,
count(*) as total_content
from new_table
group by 1;
```
## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.



## Author - Deeksha Chauda



