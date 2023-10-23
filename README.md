# Music-store-analysis
A local music store wantst to organize a music festival, for which it needs to more information, to pln out the event. There are 11 questions to be answered using the music store databse. 

# Question 1
Who is the senior most employee based on job title?

```ruby
SELECT * FROM employee
ORDER BY levels desc
Limit 1 
-- Output: Madan Mohan
```

# Question 2
Which countries have the most Invoices?

```ruby
SELECT COUNT(*) as Count, billing_country 
FROM invoice 
GROUP BY billing_country
ORDER BY count desc
LIMIT 3
-- OUTPUT: USA has 131 billings,
-- Canada has 76, Brazil has 61
```

# Question 3
What are top 3 values of total invoice?

```ruby
SELECT invoice_id, total
FROM invoice
ORDER BY total desc
LIMIT 3
-- OUTPUT: 23.759, 19.8, 19.8
```

# Question4
Which city has the best customers? We would like to throw a promotional Music
Festival in the city we made the most money. Write a query that returns one city that
has the highest sum of invoice totals. Return both the city name & sum of all invoice
totals

```ruby
SELECT SUM(total) AS invoice_total, billing_city
FROM INVOICE
GROUP BY billing_city
ORDER BY invoice_total desc
LIMIT 1 
-- Prague has the hightest sum of invoice totals(273.24),
-- thus it has the best customers
```

# Question 5
Who is the best customer? The customer who has spent the most money will be
declared the best customer. Write a query that returns the person who has spent the
most money

```ruby
SELECT customer.first_name, customer.last_name, 
SUM(INVOICE.total) AS invoice_total
FROM customer 
JOIN invoice ON customer.customer_id = invoice.customer_id
       GROUP BY customer.customer_id
       ORDER BY invoice_total desc 
--OUTPUT: R Madhav is the best customer 
--as he spends the most (144.54)
```

# Question 6
Write query to return the email, first name, last name, & Genre of all Rock Music
listeners. Return your list ordered alphabetically by email starting with A

```ruby
SELECT DISTINCT first_name, last_name, email
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
JOIN invoice_line ON invoice.invoice_id = invoice_line.invoice_id        --outer query shows values given by inner query
WHERE track_id IN(
               SELECT track_id FROM track
               JOIN genre ON track.genre_id = genre.genre_id             -- inner query brings genre name to track id
               WHERE genre.name = 'Rock')
ORDER BY email
-- Output: 59 outputs
```

# Question 7
Let's invite the artists who have written the most rock music in our dataset. Write a
query that returns the Artist name and total track count of the top 10 rock bands

```ruby
SELECT artist.name, artist.artist_id,
COUNT(artist.artist_id) AS number_of_songs
FROM track                                                         
JOIN album ON album.Album_Id = track.Album_iD
JOIN artist ON artist.artist_id = album.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name = 'Rock'
GROUP BY artist.artist_id
ORDER BY number_of_songs desc
LIMIT 10
```

# Question 8
Return all the track names that have a song length longer than the average song length.
Return the Name and Milliseconds for each track. Order by the song length with the
longest songs listed first

```ruby
SELECT name AS track_name, milliseconds AS length
FROM track
WHERE milliseconds > (SELECT AVG(milliseconds) AS average_length
					  FROM track)
ORDER BY milliseconds desc
-- 494 Outputs
```

# Question 9
Find how much amount spent by each customer on artists? Write a query to return
customer name, artist name and total spent

```ruby
--Part 1: use CTE to create a temp table 
WITH best_selling_artist AS(
     SELECT artist.artist_id AS artist_id, artist.name AS artist_name, 
            SUM(invoice_line.unit_price*invoice_line.quantity) AS total_sales
     FROM invoice_line
     JOIN track ON track.track_id = invoice_line.track_id
     JOIN album ON album.album_id = track.album_id
     JOIN artist ON artist.artist_id = album.artist_id
     GROUP BY 1
     ORDER BY 3 DESC
 )
 --Part 2: Customer details
 SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name,
 SUM(il.unit_price*il.quantity) AS amount_spent
 FROM invoice i
 JOIN customer c ON c.customer_id = i.customer_id
 JOIN invoice_line il ON il.invoice_id = i.invoice_id
 JOIN track t ON t.track_id = il.track_id
 JOIN album alb ON alb.album_id = t.album_id
 JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
 GROUP BY 1,2,3,4
 ORDER BY 5 DESC;
```

# Question 10
We want to find out the most popular music Genre for each country. We determine the
most popular genre as the genre with the highest amount of purchases. Write a query
that returns each country along with the top Genre. For countries where the maximum
number of purchases is shared return all Genres

```ruby
WITH my_cte AS (
   SELECT COUNT(invoice_line.quantity) AS total_purchases, customer.country, 
   genre.name, genre.genre_id,
   ROW_NUMBER() OVER(PARTITION BY customer.country 
					 ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo       --Use row number to get top 1 purchase of the country
   FROM invoice_line
   JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
   JOIN customer ON customer.customer_id = invoice.customer_id
   JOIN track ON track.track_id = invoice_line.track_id
   JOIN genre ON genre.genre_id = track.genre_id
   GROUP BY 2,3,4
   ORDER BY 2 ASC, 1 DESC
	          )
SELECT * FROM my_cte
WHERE RowNo <= 1
```

# Question 11
Write a query that determines the customer that has spent the most on music for each
country. Write a query that returns the country along with the top customer and how
much they spent. For countries where the top amount spent is shared, provide all
customers who spent this amount

```ruby
WITH RECURSIVE 
   customer_with_country AS (
      SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending
      FROM invoice
      JOIN customer ON customer.customer_id = invoice.customer_id
      GROUP BY 1,2,3,4
      ORDER BY 1,5 DESC),
	  
   country_max_spending AS(
      SELECT billing_country,MAX(total_spending) AS max_spending
      FROM customer_with_country
      GROUP BY billing_country)
	  
SELECT cc.billing_country,total_spending,first_name,last_name
FROM customer_with_country cc
JOIN country_max_spending ms ON cc.billing_country = ms.billing_country
WHERE cc.total_spending = ms.max_spending
ORDER BY 1;
```
