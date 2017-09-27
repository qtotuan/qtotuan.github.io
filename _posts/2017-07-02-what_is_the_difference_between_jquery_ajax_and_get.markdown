---
layout: post
title:  "What is the difference between $.ajax() and $.get()?"
date:   2017-07-02 12:11:03 +0200
categories: javascript
---
I've noticed that $.ajax() and $.get() seem to be used interchangeably. For example, when I make a request to the GitHub API to list a user, it looks like get() and ajax() are giving me the exact same thing.

Using $.get():

![get]({{ site.url }}/assets/170702_get.png)
<br><br>

Using $.ajax():

![ajax]({{ site.url }}/assets/170702_ajax.png)

<br><br>

What's the difference?

​So yes, basically both methods perform the same thing - an asynchronous HTTP request. But you could say that $.ajax() is the more 'customizable' method, while $.get() is a shorthand version.

``` javascript
$.get( url [, data ] [, success ] [, dataType ] )

// is the same as:

$.ajax({
  url: url,
  data: data,
  success: success,
  dataType: dataType
})
```
 
The most obvious difference is the syntax: you pass an object to $.ajax() and arguments to $.get(). This also means that  with $.ajax() you have more options for configuring the settings.  
Examples:
  * set custom headers and pass in API token > use 'beforeSend' to set the function
  * a username and password can be sent to an HTTP authentication request > use "username" and "password"
  * error handling - when $.get() returns an error code, it will fails silently > use 'error' to set a function that is called when the request fails
  * set the HTTP method: GET/POST/PUT > use 'method' (default: GET)

$.get() is a simplified version of $.ajax(). Many settings that usually do not need to be specified in simple requests are set to default values. For example, the data is always returned to a callback function. It also only allows GET requests. In order to perform a POST request, you would use $.post(). 

And while we are at it - what is the difference to .load()?

.load() is similar to $.get() but it also allows you to define an element where you want to place the returned HTML. It has an implicit callback function. This means that even without passing in a callback function .load() takes care of inserting the data into the matched HTML element.  
It can be used like this:  
``` javascript
$("#my-element").load("test.html")
```
This also means that .load() is only useful when the call results in HTML.

Overall, $.get(), $.post(), .load() are just wrappers for $.ajax(). The latter is always part of the former as it is called internally.
