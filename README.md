# Spotify-Advanced-SQL-and-Query-Optimization
![Spotify_Logo](Spotify-Logo.png)
This project demonstrates advanced SQL queries for Spotify data analysis, including EDA, 
(
   Artist VARCHAR(255),
   Track VARCHAR(255),
   Album VARCHAR(255),
   Album_type VARCHAR(50),
   Danceability FLOAT,
   Energy FLOAT,
   Loudness FLOAT,
   Speechiness FLOAT,
   Acousticness FLOAT,
   Instrumentalness FLOAT,
   Liveness FLOAT,
   Valence FLOAT,
   Tempo FLOAT,
   Duration_Min FLOAT,
   Title VARCHAR(255),
   Channel VARCHAR(255),
   Views_count FLOAT,
   Likes_count BIGINT,
   Comment_count BIGINT,
   Licensed BOOLEAN,
   Official_video BOOLEAN,
   Stream BIGINT,
   Energy_Liveness FLOAT,
   Most_Played_on VARCHAR(255)
);
```

---

## Steps in the Project

### 1. **Data Cleaning and Transformation**
Before performing any analysis, the dataset was first cleaned and transformed to ensure accurate and consistent results. The key steps in data cleaning included:
- **Excel for Initial Cleaning**: Removed rows with missing or inconsistent values and formatted columns for easy import into PostgreSQL.
- **SQL Cleaning**: Removed rows with 0 duration and inconsistent album names within the database.

   ```sql
   -- Delete songs with 0 duration
   DELETE FROM Spotify
   WHERE Duration_Min = 0;
   
   -- Delete rows with inconsistent album names
   DELETE FROM Spotify
   WHERE Album = '#NAME?';
   ```

### 2. **Exploratory Data Analysis (EDA)**
Once the data was cleaned, an **EDA** was performed to get an understanding of the dataset. This included counting rows, identifying distinct artists and albums, and calculating the maximum and minimum song durations. Some of the key queries include:

- **Number of Rows**: 
   ```sql
   SELECT COUNT(*)
   FROM Spotify;
   ```
  
- **Distinct Artists**: 
   ```sql
   SELECT COUNT(DISTINCT Artist)
   FROM Spotify;
   ```
  
- **Maximum and Minimum Duration**:
   ```sql
   SELECT Album, Album_type, Artist, Duration_Min
   FROM Spotify
   WHERE Duration_Min = (SELECT MAX(Duration_Min) FROM Spotify);
   ```

---

## 3. **Solving Business Problems**

### Easy Level Queries
1. **Retrieve tracks with more than 1 billion streams**:
   ```sql
   SELECT *
   FROM Spotify
   WHERE Stream > 1000000000;
   ```

2. **List all albums along with their respective artists**:
   ```sql
   SELECT DISTINCT Album, Artist
   FROM Spotify;
   ```

3. **Get the total number of comments for tracks where licensed = TRUE**:
   ```sql
   SELECT SUM(Comment_count) AS Total_No_of_Comments
   FROM Spotify
   WHERE Licensed = TRUE;
   ```

4. **Find all tracks that belong to the album type 'single'**:
   ```sql
   SELECT *
   FROM Spotify
   WHERE Album_type = 'single';
   ```

5. **Count the total number of tracks by each artist**:
   ```sql
   SELECT Artist,
          COUNT(*) AS Total_No_of_Tracks
   FROM Spotify
   GROUP BY Artist;
   ```

---

### Medium Level Queries
1. **Calculate the average danceability of tracks in each album**:
   ```sql
   SELECT Album,
          AVG(Danceability) AS Average_Danceability
   FROM Spotify
   GROUP BY Album;
   ```

2. **Find the top 5 tracks with the highest energy values**:
   ```sql
   SELECT DISTINCT Track, Album, Artist, Energy
   FROM Spotify
   ORDER BY Energy DESC
   LIMIT 5;
   ```

3. **List all tracks along with their views and likes where official_video = TRUE**:
   ```sql
   SELECT Track,
          SUM(Views_count) AS Views, 
          SUM(Likes_count) AS Likes
   FROM Spotify
   WHERE official_video = TRUE
   GROUP BY Track
   ORDER BY Likes DESC;
   ```

4. **For each album, calculate the total views of all associated tracks**:
   ```sql
   SELECT Album,
          SUM(Views_count) AS Total_Views
   FROM Spotify
   GROUP BY Album;
   ```

5. **Retrieve the track names that have been streamed on Spotify more than YouTube**:
   ```sql
   SELECT *
   FROM (SELECT Track,
                SUM(CASE WHEN Most_Played_on = 'Spotify' THEN Stream ELSE 0 END) AS Stream_Spotify,
                SUM(CASE WHEN Most_Played_on = 'Youtube' THEN Stream ELSE 0 END) AS Stream_Youtube
         FROM Spotify
         GROUP BY Track) AS t 
   WHERE Stream_Spotify > Stream_Youtube 
     AND Stream_Youtube <> 0;
   ```

---

### Advanced Level Queries
1. **Find the top 3 most-viewed tracks for each artist using window functions**:
   ```sql
   WITH ranking_artist AS (
     SELECT Artist, 
            Track, 
            SUM(Views_count) AS Total_Views,
            DENSE_RANK() OVER (PARTITION BY Artist ORDER BY SUM(Views_count) DESC) AS rank
     FROM Spotify
     GROUP BY Artist, Track
   )
   SELECT *
   FROM ranking_artist
   WHERE rank <= 3;
   ```

2. **Find tracks where the liveness score is above the average**:
   ```sql
   SELECT *
   FROM Spotify
   WHERE Liveness > (SELECT AVG(Liveness) FROM Spotify);
   ```

3. **Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album**:
   ```sql
   WITH EnergyLevel AS (
     SELECT Album,
            MAX(Energy) AS Max_Energy,
            MIN(Energy) AS Min_Energy
     FROM Spotify
     GROUP BY Album
   )
   SELECT Album, 
          (Max_Energy - Min_Energy) AS Difference
   FROM EnergyLevel
   ORDER BY Difference DESC;
   ```

4. **Find tracks where the energy-to-liveness ratio is greater than 1.2**:
   ```sql
   SELECT Track,
          (Energy / Liveness) AS Energy_Liveness_Ratio
   FROM Spotify
   WHERE Energy / Liveness > 1.2
   ORDER BY Energy_Liveness_Ratio DESC;
   ```

5. **Calculate the cumulative sum of likes for tracks ordered by the number of views using window functions**:
   ```sql
   SELECT Artist, 
          Track, 
          Views_count, 
          Likes_count,
          SUM(Likes_count) OVER (ORDER BY Views_count DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS Cumulative_Likes
   FROM Spotify
   ORDER BY Views_count DESC;
   ```
## QUERY OPTIMIZATION
### What is SQL Query Optimization?

SQL query optimization is the process of enhancing the performance of SQL queries by improving their execution speed and minimizing the resources they consume. This involves refining how queries are structured and executed so that they retrieve the required data quickly and efficiently. Effective optimization ensures that the database system handles large amounts of data without slowing down, improving overall system performance.

### Advantages of SQL Query Optimization

1. **Improved Performance**: Faster query execution results in quicker access to data.
2. **Resource Efficiency**: Optimized queries reduce CPU, memory, and storage usage, leading to lower operational costs.
3. **Better User Experience**: Shorter response times enhance the user experience by reducing wait times for data retrieval.
4. **Scalability**: Optimized queries allow the system to handle more data and users without significant performance degradation.

### Best Practices for SQL Query Optimization

1. **Use Indexes**
2. **Use WHERE Clause Instead of HAVING**
3. **Avoid Queries Inside a Loop**
4. **Use SELECT Instead of SELECT * .**
5. **Add EXPLAIN to the Beginning of Queries**
6. **Keep Wildcards at the End of Phrases**
7. **Use EXISTS() Instead of COUNT()**
8. **Avoid Cartesian Products**
9. **Consider Denormalization**
10. **Optimize JOIN Operations**
### EXAMPLE
**BY USING INDEXES**
## Initial Query Performance Analysis
We performed an initial analysis of a query using `EXPLAIN ANALYZE` to retrieve the top 5 tracks by **Taylor Swift** from a Spotify table, ordered by the number of streams in descending order. Below are the performance metrics for the initial query:

- **Execution Time (E.T.):** 4.76 ms  
- **Planning Time (P.T.):** 0.143 ms

![Before Query Optimization](Before_Query_Optimization.png)

## Optimization: Index Creation on Artist Column
To improve performance, we created an index on the **Artist** column, significantly enhancing the efficiency of retrieving rows where the artist is queried. The following SQL command was used to create the index:

```sql
CREATE INDEX idx_artist ON spotify_tracks(artist);
```

## Post-Optimization Performance
After creating the index, we re-executed the query and observed a **substantial improvement** in performance:

- **Execution Time (E.T.):** 0.153 ms  
- **Planning Time (P.T.):** 0.152 ms

![After Query Optimization](After_Query_Optimization.png)

This demonstrates that the index greatly reduced both execution and planning time, resulting in a highly optimized query.


---

## Conclusion
This project demonstrates how **Excel** and **SQL** can be used to clean, transform, and analyze large datasets like Spotify's track listings. Through exploratory data analysis and solving various business questions, the project provides insights into music trends, streaming patterns, and song popularity, while utilizing advanced SQL techniques such as window functions and aggregate operations. Optimizations were implemented throughout to ensure the queries performed efficiently.

Feel free to explore the SQL queries and customize them for your own dataset analysis!

---

### Tools and Technologies Used:
- **Excel** for initial data cleaning
- **PostgreSQL / pgAdmin4** for SQL querying
- **SQL** for data analysis
- **Kaggle** for the dataset

### Dataset:
The dataset used for this project is sourced from Kaggle, which includes various attributes of Spotify tracks such as artist, track name, album, streams, and more. You can access the dataset [here](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset).
