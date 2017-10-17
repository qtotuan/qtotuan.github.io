---
layout: post
title:  "Thoughts on A/B testing"
date:   2017-10-17 12:11:03 +0200
categories: jekyll update
---
Today I am pondering something that is at the same time old and new to my brain - A/B testing. In my former life as a digital marketer I would test Facebook ads, version A vs version B, to see which one would generate the higher conversion rate. Implementing tests was extremely convenient - all I needed to do was to upload the ads onto the demand side platform, run two separate campaigns and look at the resulting data.

In my post-marketing life it takes a bit more than just a button click to implement a test. So my question today is:  

How does a web developer implement A/B testing for a web application?

Let's think through this example: We have a Ruby on Rails app, and we'd like to test which heading on the homepage is more popular with the user.

For illustration purposes, we have these two alternative headings:

``` html
<h1>You look awesome today!</h1>
```

``` html
<h1>Scientists confirm: A book a day keeps the doctor away.</h1>
```

Now, we need to generate a random test variable that decides which header to render. It could look something like this:

``` ruby
# Generate a random number, either 0 or 1
random = rand(2)
if random == 0
  header = "Test A"
else
  header = "Test B"
end
```

The question that arises is as follows:


## WHERE TO GENERATE THE RANDOM TEST VARIABLE?

Let's walk through the basic process of how a page is rendered with Rails. The user types in the URL into the browser and hits enter. This generates a GET HTTP request, which hits the controller method #index. This method renders the index.erb file, which contains the HTML of our page.

So now we have two potential candidates for our random-logic: #index and index.erb. If we were to generate the random variable in the controller, we can have access to it through the erb template. The controller method would contain something like this:

``` ruby
# controller
def index
  test_version = rand(2)
  if test_version == 0
    @test_header = "You look awesome today!"
  else
    @test_header = "Scientists confirm: A book a day keeps the doctor away."
  end
end
```

In the view we'd simply output the tag like so:

``` erb
# index.erb
<h1><%= @test_header %></h1>
```

However, it seems more natural to keep the controller clean of concrete outputting data. So we'll generate the variable in the controller, but move the conditional logic with the two header to the view.

``` ruby
# controller
def index
  @test_version = rand(2)
end
```

``` erb
# index.erb
<% if @test_version == 0 %>
  <h1>You look awesome today!</h1>
<% else %>
  <h1>Scientists confirm: A book a day keeps the doctor away.</h1>
<% end %>
```

Now, there is another important point to take into consideration. What we want to do is keep the headers consistent for a user. After a first time visit, how do we make sure that upon following visits the user gets the same header? Consequently, the test variable needs to be stored somewhere.


## WHERE TO STORE THE TEST VARIABLE?

We have two options: either in the front end or back end.

If we store the data in the back end, it would mean that we are hitting our database. We'd want to store something like: user id -> test variable. But how would we identify the user/client? How and what kind of information would we pass from front to back end? It seems more logical and straight forward to store the variable on the client side.

We can make use of multiple ways to store data on the client side:
* cookies
* Window.localStorage
* Window.sessionStorage

What is the difference?

Local storage and session storage are two objects of the HTML web storage. While the local storage keeps data without an expiration date, the session storage data is deleted when the browser is closed. So the data in the local storage is much longer persistent and gets only cleared through JavaScript or by clearing the browser cache. And before HTML5 was released, data was stored in cookies, which was sent along in any subsequent HTTP request. Its expiration date varies - it can be set from the server or client side.

Another difference to note is that web storage can only be read on the client side, while cookies are mainly for server side reading (they can also be read on the client side), as they are sent in requests.


## COOKIES OR WEB STORAGE?

It is helpful to consider further questions pertaining to our test goals in order to narrow down the selection:

1. How long is the test going to last? A few hours, days, weeks, months?
In our example case, we can assume that we want to test the headers over multiple days or weeks, rather than just a few hours, so it makes more sense to favor the more persisting options of local storage and cookies over session storage.

2. Who needs the data - client side or server side?
The logic for the random test variable lives in our erb template, on the server side, so this is where the information is needed.
If we decided to use the local storage, we would purposely use JavaScript on the front end to get the item from the web storage. Then the retrieved data can be sent within the header of the request to the back end.
A cookie is sent with every request. With local storage, on the other hand, we can be more selective with when to send the data.  In our example test case, our data is not that large that it would make a huge difference in bandwidth. It were a different case if our app were e.g. an e-commerce shop, where we would store information on multiple products for the user's shopping cart. Sending this info with each interaction would have a more noticeable impact on the site performance.
** Excursion: If our front end were built on React, the web components would be computed on the client side. Thus the random test variable would not need to be sent to the server side.


## SUMMARY
To tie it all together, we have multiple options for generating and storing the random test variable. This is the flow for the Rails app example using cookies:

* The server side is generating the test variable. The controller generates the random variable and the erb template uses it to render the according header.
* At the same time, it creates a cookie (if none is existent) and stores the test variable in it. This cookie is sent to the client side in the HTTP response and sits on the user's local machine.
* Every time the user returns to the page the cookie is sent along with the HTTP request to the server. The controller reads the cookie with the test variable and based on this renders the same header.
