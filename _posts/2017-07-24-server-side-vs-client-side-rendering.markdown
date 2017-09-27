---
layout: post
title:  "Server Side vs. Client Side Rendering"
date:   2017-07-24 12:11:03 +0200
categories: javascript
---
## Why does this even matter?

Consider websites like Amazon.com. They claim that a millisecond of latency, where the user has to wait for a page load, cost them millions of dollars. Or Twitter, who have tons of fresh data coming in every second that needs to be rendered fast on any mobile device.
These companies need to carefully consider how they architect their websites and apps, because slow performance means bye-bye money.


## What is the difference between Server Side and Client Side Rendering?

Back then when the internet started, all that was expected was a page with information to render.  All the user wanted to do (and could) was look at static text and images. The traditional way of displaying the site on the screen was to ask the the server to compile the HTML and send it to the browser. No problem. Rendering static files is simple and cheap. This is Server Side Rendering - SSR.

Then along came a whole new level of webpages - sites showed off revolutionary elements like dropdown menus, pictures in gallery sliders, date pickers, send messages, etc. The web became interactive! Actually many webpages are more like apps than sites. JavaScript hooray! But wait a minute - reloading the whole page for updating one piece of data suddenly appears very clunky and slow. Who wants to see ten loading animations every time they click on something? And re-rendering the whole page for one little change seems just exaggerated and inefficient in terms of operating time.

Client Side Rendering to the rescue! What if the page load was split into two parts. The HTML template and the JS files that produce the HTML elements programmatically? The whole page would load completely initially and after that would only have to change the HTML bits which require new data.

In summary.

SSR:
The browser makes one HTTP request ad receives the full HTML document. The browser immediately renders the page and it is immediately viewable. If the user visits a different page on the website, the browser will once again go through this process and make another request for a new HTML document.

CSR:
The browser makes two requests (not only one). The first one will return HTML and CSS files. But the HTML file is empty, it is a document with links to the javascript files. The browser then makes a second request to retrieve the JS files. When those are downloaded they can be executed and then the page is viewable and interactive. So CSR means that content is being rendered in the browser using JavaScript.


## What are the Pros and Cons?

<span style="color:blue">Initial load time</span>  
SSR initial page load is faster than CSR, as the latter has to make two HTTP requests before the page can be rendered. SSR does not have to wait for javascript to download before it can render the page. The user can see the content sooner. How fast the request is processed depends on multiple factors, for example:
  * Internet speed
  * Location of the server
  * Number of users trying to access the site
  * File size  

This is why with CSR the risk is higher that the user sees a blank page or a loading animation especially if the user has a slow internet connection, or the app files contain heavier components.  
=> Winner: <span style="color:green">SSR</span>


<span style="color:blue">Interactivity</span>  
With CSR, the initial load is heavy but after that the page only needs to rebuild the interactive parts. Instead of re-rendering the entire page, only the elements that changed need to be updated. No need to make a new server request (except new data is needed). This makes the page feel very snappy. With SSR the server has to execute a server request and full page reload for every user interactivity. This can lead to highly increased loading times and less responsiveness. The advantage becomes alot more pronounced when the users is on his mobile device without Wi-Fi.  
=> Winner: <span style="color:green">CSR<span>

<span style="color:blue">SEO</span>  
The Google crawlers cannot index CSR loaded blank pages. With SSR the full HTML of the page is immediately crawlable, but when using CSR the parts that make up the page are separated. There are ways around it but there is no magic bullet and getting SEO to work for CSR is much more of a hassle than with SSR.  
=> Winner: <span style="color:green">SSR</span>


## When to use SSR and when to use CSR?
Use SSR when
 * the website is mostly static, does not require rich animation and much interactivity
 * SEO really matters on all search engines

Use CSR when
* the website is very dynamic: it fells more like an app than a page
* SEO matters less on Bing, Yahoo, Baidu


## What is isomorphic?
A new age has dawned. Apps that are isomorphic combine the best of both worlds: they are made up of code that can run both in the server AND client. JavaScript has traditionally ran on the client side but with Node.js has become a server side language as well.
The first request from the browser is processed by the server: it receives and renders the the full HTML on initial page load and displays it immediately. All other subsequent requests are processed by the client.
​

Helpful sources:
https://medium.freecodecamp.org/what-exactly-is-client-side-rendering-and-hows-it-different-from-server-side-rendering-bd5c786b340d
http://andrewhfarmer.com/server-side-render/
https://blog.webix.com/client-side-vs-server-side-ui-rendering/
https://www.lullabot.com/articles/what-is-an-isomorphic-application
