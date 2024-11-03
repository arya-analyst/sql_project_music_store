# sql_project_music_store

## Database and Tools
- Oracle
- SQL Developer

## Schema
![Schema](https://github.com/arya-analyst/sql_project_music_store/blob/main/schema_diagram.png)

### Q1. Who  is the senior most employee based on job title?
```sql
Select * from employee
order by levels desc
Fetch FIRST 1 Row ONLY;

-- Alternate

Select * from employee
Where reports_to IS NULL;

```
### Q2. Which countries have the most invoices?
```sql
Select billing_country, count(*) as NOI
from invoice
group by billing_country
order by noi desc;

-- Alternate

Select Unique billing_country,
count (*) over (partition by billing_country ORDER BY billing_country desc ) as NOI
from invoice
Order By NOI DESC
```

### Q3. Which city has the best customers? We would like to throw a promotional music festival in the city we made the most money. Write a query that returns one city that has the highest sum of invoice totals.
```sql
Select billing_city, round(sum (total),2) as invoice_total
from invoice
group by billing_city
order by invoice_total desc;
```

### Q4. Which customer spent the most? 
```sql
select c.customer_id, c.first_name, trunc(sum(i.total),2) as total_spent
from customer c
join invoice i
on c.customer_id = i.customer_id
group by c.customer_id, c.first_name
order by total_spent desc
FETCH FIRST 1 ROW WITH TIES;
```

### Q5. Write a query to return the email, name and genre of all rock music listeners. Return your list ordered alphabetically by email starting with A.
```sql
Select UNIQUE c.first_name || ' ' || c.last_name as name, c.email, g.name as genre_name
from customer c
Join invoice i on c.customer_id = i.customer_id
Join invoice_line il on i.invoice_id = il.invoice_id
Join track t on il.track_id = t.track_id
Join genre g on t.genre_id = g.genre_id
Where t.genre_id = 1
order by c.email asc


-- Alternate

Select DISTINCT c.first_name || ' ' || c.last_name as name, c.email
from customer c
Join invoice i on c.customer_id = i.customer_id
Join invoice_line il on i.invoice_id = il.invoice_id
    Where il.track_id IN (
        Select track_id from track
        Join genre on track.genre_id = genre.genre_id
        Where genre.name LIKE '%Rock%' )
Order By email asc;
```

### Q6. Let's invite the artists who have written the most rock music in our dataset. Write a query that returns the artist name and total track count of the 10 rock bands
```sql
Select artist.artist_id, artist.name, count (artist.name) as no_of_songs from artist
join album on artist.artist_id = album.artist_id
join track on album.album_id = track.album_id
Where track.genre_id = 1
Group By artist.artist_id, artist.name
Order By no_of_songs desc
Fetch First 10 ROWS WITH TIES;
```

### Q7. Return all the track names that have a song length longer than the average song length. Return the Name, time duration for each track. Order by the song length with the longest songs listed first.
```sql                 
Select name, round((milliseconds/60000),2) as time_duration
from track
Where round((milliseconds/60000),2) > (Select round(avg(milliseconds/60000),2) as avg_time from track)
Order By time_duration desc

-- Alternate 

Select name, milliseconds
from track
where milliseconds > (
                        Select avg(milliseconds) as avg_track_length
                        From track )
order by milliseconds desc
```

### Q8. We want to find out the most popular music genre for each country. We determine the most popular genre with the highest amount of purchases. Write a query that returns each country along with the top genre.
```sql
With popular_genre AS 
(
    Select i.billing_country, count (il.quantity) as purchases, g.genre_id, g.name,
    dense_rank () over (partition by i.billing_country order by count (il.quantity) desc ) as rank
    from invoice i
    Join invoice_line il on i.invoice_id = il.invoice_id
    Join track t on il.track_id = t.track_id
    Join genre g on t.genre_id = g.genre_id
    group by i.billing_country, g.genre_id, g.name
    order by i.billing_country asc, rank asc 
)

Select billing_country as country, purchases, name as genre_name from popular_genre where rank = 1;
```

### Q9. Write a query that determines the customer that has spent the most on music for each country. Write a query that returns the country along with the customer and how much do they spent
```sql
With customer_total_spending AS
(
    Select c.customer_id, c.first_name, c.last_name, i.billing_country, sum(i.total) as total_spent,
    dense_rank () over(partition by i.billing_country order by sum(i.total) desc ) as dr
    From customer c
    Join invoice i ON c.customer_id = i.customer_id
    GROUP BY i.billing_country,c.customer_id, c.first_name, c.last_name
    Order By i.billing_country, dr, total_spent
)

Select customer_id, first_name, last_name, billing_country as country, trunc(total_spent, 2) as total_spent
From customer_total_spending Where dr = 1
```
