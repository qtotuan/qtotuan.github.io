---
layout: post
title:  "How to Get the Current Week in Javascript"
date:   2017-09-21 12:11:03 +0200
categories: javascript
---
Currently I am working on my final project for my coding bootcamp - a Habit Tracker! And in this habit tracker the user will be able to click on a date of the current week to mark the habit as "done".

So - How do I get the dates of the current week? Let's say it is Saturday and the user should see the dates from Monday through Sunday.


![Calendar Week]({{ site.url }}/assets/calendar-week.png)

 

In a rare optimistic moment I googled "javascript current week method", hoping that I would find something like 'date.currentWeek()'.

Fail.

It looked like I had to write a Javascript function from scratch. I found a nice StackOverflow thread that helped me find a solution. And on the way, I learnt some cool built in Javascript date methods.

This brings me to a point. How often have I heard ever since I started this bootcamp: "You do not have to reinvent the wheel". There is a multitude of problems that people have already solved. It is not worth spending hours racking your brain to come up with your own solution and "oversolve" this issue. Except you are seeing great benefit for your learning curve in this particular area. Other than that: just google. Just stackoverflow.

Here we go. Today is Saturday. My goal is to create an array of all the dates from this current week, Monday through Sunday.

What we need to do is get the day of the month, find out the weekday (Tuesday, Wednesday, etc), subtract the number needed to get to Monday, and then create the new date using the new calculated day and combine it with the current month. We are going to make use of Javascript's built in methods: new Date, getDate, getDay, and setDate. Got it? 

It seems wild but it will make more sense when we go through the code.

{% highlight javascript linenos %}

let curr = new Date
let week = []

for (let i = 1; i <= 7; i++) {
  let first = curr.getDate() - curr.getDay() + i
  let day = new Date(curr.setDate(first)).toISOString().slice(0, 10)
  week.push(day)
}

{% endhighlight %}

Line 1: [new Date](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Date)  
=> This gives us todays' date, i.e. "Sat Aug 12 2017 15:22:48 GMT-0400 (EDT)"

Line 5: [getDate()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getDate)  
=> Returns the day of the month, in this case "12"

Line 5: [getDay()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getDay)  
=> This is where the magic happens!  
=> Returns the day of the week.  
=> The return value is an integer, where Sunday = 0, Monday = 1, Tuesday = 2, etc.!  
=> In our case it returns 6 for Saturday  
=> This allows us to subtract this return value from today's day of the month so that we get Monday's day of the month. To get to Monday we will subtract 6 + 1. We need the +1 or else we'd land at Sunday.  
=> 12 - 6 + 1 = 7  
=> Aug 7th is a Monday  
=> BAM!

Then it is only a matter of looping for each day of the week and push each date into my week's array. But we also need to reconstruct our date.

Line 6: [setDate()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/setDate)  
=> sets the day of the Date object relative to the beginning of the currently set month  
=> We need this method in order to construct our real date. So far first is "7" and we need to combine it with the current month to get to Aug 7th.  
=> The return value is something weird like this: 1502134686498. In case you are wondering. This is the number of milliseconds between 1 January 1970 00:00:00 UTC and the given date.  
=> This is why we need to wrap this into "new Date", to get to our desired date format  
=> new Date(1502134686498) returns "Mon Aug 07 2017 15:38:06 GMT-0400 (EDT)"

I've used some additional formatting due to my app's needs. 

Line 6: [toIsoString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString)  
=> returns "2017-08-07T19:38:06.498Z"  
=> And because I only needed the date without time, I sliced my string accordingly.

And here we go, my final "week" array is done!  
=> ["2017-08-07", "2017-08-08", "2017-08-09", "2017-08-10", "2017-08-11", "2017-08-12", "2017-08-13"]


Phew, that was a nice little brain teaser! For my app, it did not end there. As I need to compare the Javascript date strings with my Ruby objects in my Rails database, I needed to add more conversions. But this is a topic for a separate blog post... 
