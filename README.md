# Spotify Advanced SQL Project and Query Optimization P-6

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```
## Project Steps

### 1. Data Exploration
```sql
SELECT COUNT(*) FROM SPOTIFY;


DELETE FROM SPOTIFY 
WHERE DURATION_MIN=0
```

## BUSSINESS PROBLEMS
```sql

--1.Retrieve the names of all tracks that have more than 1 billion streams.

SELECT * FROM SPOTIFY 
WHERE stream > 1000000000;
```
```sql

--2.List all albums along with their respective artists.
select album, artist from spotify;
```
```sql

--3.Get the total number of comments for tracks where licensed = TRUE.

select sum(comments) from spotify where licensed='true';
```
```sql

--4.Find all tracks that belong to the album type single.

select track  from spotify where album_type='single';
```
```sql

--5.Count the total number of tracks by each artist.

select * from spotify;

select artist, count(track) from spotify group by 1 order by 2 desc;
```

```sql

--6.Calculate the average danceability of tracks in each album.


select album, avg(danceability) from spotify group by 1 order by 2;
```
```sql

--7.Find the top 5 tracks with the highest energy values.

select track, energy from spotify order by 2 desc limit 5;
```
```sql

--8.List all tracks along with their views and likes where official_video = TRUE.

select track,sum(views) as total_views,sum(likes) as total_likes from spotify where official_video = 'true' group by 1;
```
```sql

--9.For each album, calculate the total views of all associated tracks.

select album, track, sum(views) from spotify group by 1,2;
```
```sql

--10.Retrieve the track names that have been streamed on Spotify more than YouTube.
select * from 
(select track,
coalesce(sum(case when most_played_on = 'Youtube' then stream end),0) as streamed_on_yt,
coalesce(sum(case when most_played_on = 'Spotify' then stream end),0) as streamed_on_spotify
from spotify
--where streamed_on_spotify > streamed_on_yt
group by 1) as t1
where streamed_on_spotify > streamed_on_yt
and streamed_on_yt<>0
```
```sql

--11.Find the top 3 most-viewed tracks for each artist using window functions.

with ranking_artist
as
(
select artist,
track,
sum(views) as total_views,
dense_rank() over(partition by  artist order by sum(views) desc ) as rank
from spotify 
group by 1,2
order by 1,3 desc
)
select * from ranking_artist
where  rank<=3
```
```sql


--12.Write a query to find tracks where the liveness score is above the average.
SELECT track, artist, liveness
FROM spotify
WHERE liveness > (SELECT AVG(liveness) FROM spotify);
```

```sql

--13.Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.

with cte
as
(
select album,
max(energy) as highest_energy,
min(energy) as lowest_energy
from spotify 
group by 1
)
select album, highest_energy-lowest_energy as diff_energy 
from cte
order by 2 desc
```
```sql

--14.Find tracks where the energy-to-liveness ratio is greater than 1.2.


SELECT track, artist, energy, liveness, (energy / liveness) AS energy_to_liveness_ratio
FROM spotify
WHERE (energy / liveness) > 1.2;
```
```sql

--15.Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.


SELECT 
    track, 
    artist, 
    views, 
    likes, 
    SUM(likes) OVER (ORDER BY views) AS cumulative_likes
FROM spotify
ORDER BY views;
```



