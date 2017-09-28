---
layout: post
title: "Can I use MAX(COUNT()) in SQL?"
date: 2017-06-04 12:11:03 +0200
categories: sql
---
I came across an interesting SQL challenge that looked easy first and then proved to be a bit tricker than expected.

And the short answer to the above question is, no. You can't. It is not possible to nest aggregate functions.

So, I have a list with all the guests that appeared on the DailyShow from 1999 - 2015. The dataset includes the date/year of the show, guest name, and their occupation.

The question is:
What was the most popular profession of guest for each year Jon Stewart hosted the Daily Show?


This is the table schema:

 ```
 CREATE TABLE dailyshow (
  id INTEGER PRIMARY KEY,
  year TEXT,
  occupation TEXT,
  show_date TEXT,
  category TEXT,
  name TEXT);
```

First, let's find out the most common occupation over ALL the years:

```
SELECT occupation, COUNT(occupation)
FROM dailyshow
GROUP BY occupation
ORDER BY COUNT(occupation) DESC LIMIT 1;
```

Easy enough. The result:

![result1]({{ site.url }}/assets/170604_result1.png)

So far so good. Now, how do we get this result by year?

To start, let's get the occupation count by year for each profession:

```
SELECT year, occupation, COUNT(occupation)
FROM dailyshow
GROUP BY year, occupation
ORDER BY year, COUNT(occupation) DESC;
```

The result is the occupation count for each occupation for each year:

![result2]({{ site.url }}/assets/170604_result2.png)


Now how do we limit these results for each year to the highest occupation count?
We'd expect to use MAX on COUNT(occupation) and add the year to the GROUP BY. Something like this:

```
SELECT occupation, MAX(COUNT(occupation))
FROM dailyshow
GROUP BY year, occupation;
```

Uh-oh.

![error]({{ site.url }}/assets/170604_error.png)

Hm, another try is to use a WHERE clause:

```
SELECT occupation, COUNT(occupation)
FROM dailyshow
WHERE COUNT(occupation) = MAX(COUNT(occupation))
GROUP BY occupation;
```

Nope.

![error]({{ site.url }}/assets/170604_error.png)

So how can we find an alternative to the MAX(COUNT()) that we are dreaming of?

The solution is to use the first table as a subquery. We will create an additional query, an outer query, which uses the first table in its FROM clause. It will be able to use MAX() on the COUNT() result from the first table, thus circumventing the direct use of two layered aggregate functions.

1./ Inner query > Reminder, this is how it looked like:

```
SELECT year, occupation, COUNT(occupation)
FROM dailyshow
GROUP BY year, occupation
ORDER BY year, COUNT(occupation) DESC;
```

2./ Outer Query > Now we are creating an ALIAS "occupation_count" for COUNT(occupation) column in the inner query so that we can reference it from the outer query.

```
SELECT year, occupation, MAX(occupation_count)
FROM (SELECT year, occupation, COUNT(occupation)  AS occupation_count
FROM dailyshow
GROUP BY year, occupation
ORDER BY year, occupation_count DESC)
GROUP BY year;
```

And voila, there we have it:

![result3]({{ site.url }}/assets/170604_result3.png)
<br><br>

To sum it up:  
The problem was that it is not allowed to use two aggregate functions in the SELECT statement. The solution is to create a table to get the results from the inner function first, and then use this table for calling the second function on the respective column.

<br>
Data can be found on [github](https://github.com/qtotuan/bonus-sql-challenge-web-051517).
