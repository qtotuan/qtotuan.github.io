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

Now, we'd have some logic that would randomize the choice of CTA. We'd generate a random variable that holds the CTA for this user. It could look something like this:

``` ruby
random = rand(10)
if random <= 5
  @cta = "Test A"
else
  @cta = "Test B"
end
```

Where can we potentially implement this logic? On the front end? In the back end? In the controller? In the view?

Let's walk through the basic process of how a page is rendered. The user types in the URL into the browser and hits enter. This generates a GET HTTP request, which hits the controller method #index. This method renders the index.erb file, which contains the HTML of our page.

So now we have two potential candidates for our random-logic: #index and index.erb. If we were to generate the random variable in the controller it could would look like this:

``` ruby
def index
  random = rand(10)
  if random <= 5
    @cta = "Test A"
  else
    @cta = "Test B"
  end
end
```

In the view of index.erb we'd have:

```
<% if @cta == "Test A" =>
  <%= <h1>You look awesome today!</h1> %>
<% else %>
 <%= <h1>Scientists confirm: A book a day keeps the doctor away.</h1> %>
<% end %>
```
