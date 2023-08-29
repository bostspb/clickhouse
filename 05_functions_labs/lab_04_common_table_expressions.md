# Lab 4. Common Table Expressions

## Task 1
Write a query on the `amazon_reviews` table that returns the average number of helpful votes for products 
in the 'Music' category. Use a CTE to define 'Music' as the category.
> Dataset `amazon_reviews` was broken down, so I used `reddit` and change a task.

```sql
WITH '1000words' AS sr
SELECT 
    avg(score)
FROM reddit
WHERE subreddit = sr;

-- 2.018324607329843
```

## Task 2
Write a query that counts the number of reviews in `amazon_reviews` where the number of helpful votes of the review 
is greater than or equal to the overall average of helpful votes.
> Dataset `amazon_reviews` was broken down, so I used `reddit` and change a task.
```sql
WITH (
    SELECT avg(score) FROM reddit
) AS avg_score
SELECT 
    subreddit, 
    avg(score) sr_avg_score 
FROM reddit
WHERE score >= avg_score
GROUP BY subreddit
ORDER BY sr_avg_score DESC
LIMIT 10;
```
```
bestof2009      93.498224852071
DateRape        63.83756906077348
latestpoll      60
announcements   33.06200558399255
gnu             29.48310810810811
GamingTourney   26.765432098765434
raerth          26.566037735849058
snowpics        24.88888888888889
lego            24.50204081632653
mspainttoday    24.25581395348837
```