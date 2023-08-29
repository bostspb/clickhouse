# Lab 2. Using Functions
> Dataset `amazon_reviews` was broken down, so I used `hackernews` from Lab 2 Lesson 2

## Task 1
Write a query that returns the number of reviews per year. 
Use the `toYear` function to convert the date to a year.
```sql
SELECT
    count(),
    toYear(time) AS year
FROM hackernews
GROUP BY year;
```
```
18      2016
58      2017
134     2018
161     2019
471     2020
438     2021
```

## Task 2
Write a query that extracts the tokens (the individual words) of the `text` column of `hackernews` table 
into an array of Strings. Limit the query to 100 rows.
```sql
SELECT splitByChar(' ', text)
FROM hackernews
LIMIT 5;
```
```
["June","(YC","W21)","|","Founding","Engineer","|","Remote","|","Full-time<p>June","(<a","href=\"https:&#x2F;&#x2F;june.so\"","rel=\"nofollow\">https:&#x2F;&#x2F;june.so</a>)","is","instant","product","analytics.","We","connect","to","Segment","and","automatically","generate","graphs","of","the","metrics","companies","should","track.<p>-","Join","a","team","of","3<p>-","Product","minded,","talk","with","users,","write","code,","scale","the","infrastructure<p>-","Interesting","challenges","(like","scaling","a","Clickhouse","cluster","and","writing","a","DSL","for","filtering","user","cohorts)<p>Learn","more:","<a","href=\"https:&#x2F;&#x2F;www.notion.so&#x2F;Founding-Engineer-339274009f594b58aff3d4bfd8e3f93e\"","rel=\"nofollow\">https:&#x2F;&#x2F;www.notion.so&#x2F;Founding-Engineer-339274009f594b58aff3...</a><p>If","interested","reach","out","at","work","[at]","june","[&#x2F;dot]","so"]
["We&#x27;re","planning","to","deploy","ClickHouse","from","Yandex[0].","Would","like","to","hear","from","anyone","who","has","it","in","production","already,","and","what","is","your","experience","with","it.<p>[0]","<a","href=\"https:&#x2F;&#x2F;clickhouse.yandex&#x2F;\"","rel=\"nofollow\">https:&#x2F;&#x2F;clickhouse.yandex&#x2F;</a>"]
["Or","normalize","your","data","and","use","clickhouse."]
["Been","there","done","that.<p>Selecting","a","database","is","the","least","of","your","worries.","And","you","learn","to","live","with","the","limitations","-","for","example,","in","clickhouse","you","add","a","millisecond","and","a","microsecond","field.<p>Litteraly","every","solution","listed","will","work,","as","the","database","will","only","be","used","to","persist","your","data.","Your","trading","will","NOT","use","your","database","in","any","way","except","to","load","the","data","when","you","start","(and","potentially","restart)","your","bots.<p>What","really","matters","is","1)","execution,","2)","network","performance","and","finally","3)","good","data<p>Language,","database","etc","are","just","tools.","This","is","a","case","of","premature","optimization."]
["&gt;","ClickHouse","come","to","mind","for","wholly","different","use","cases","than","a","time","series","database.<p>ClickHouse","works","fine","as","a","TSDB","if","you","don&#x27;t","mind","getting","a","little","dirty"]
```

## Task 3
Using the previous query, write a query that returns the number of reviews where the length of the review 
has at least 100 words. (This query may take a while based on your available resources).
```sql
SELECT count()
FROM hackernews
WHERE length(splitByChar(' ', text)) >= 100;
-- 304
```

## Task 4
Which author writes the most reviews? (Return the `by`.)
```sql
SELECT 
    by,
    count() AS number_of_reviews
FROM hackernews
GROUP BY by
ORDER BY number_of_reviews DESC
LIMIT 1;

-- hodgesrm	78
```

## Task 5
How many different authors are there?
```sql
SELECT uniq(by) 
FROM hackernews;

-- 557
```

## Task 6

## Task 7

## Task 8

## Task 9

## Task 10


