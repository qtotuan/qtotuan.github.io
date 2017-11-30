---
layout: post
title:  "A/B Test II"
date:   2017-10-30 12:11:03 +0200
categories: javascript
---

* How we'd analyze performance of an AB-test? For this we need to store users test versions and their interaction (clicks / signups / purchases) with respect to their test version. This thing affects any AB-test setup a lot.

So we'd need to get the data from the frontend to the backend somehow. For example, how would I store click count according to test version? I could place event handlers on a button that triggers an Ajax call. There are 2 possible variations:
Version 1: This call will trigger an HTTP Get request. This request calls a landing page with a certain end point, according to the test version (from the user's cookie). This site is merely there to record a click (whenever called, it will increase its count in the database, something like document.onLoad) and immediately redirect back to the original page. The user will see a short flicker on reload.
Version 2: This call will trigger an HTTP PATCH request. It sends the test version via the cookie in this call. It hits the controller's method to increase the count in the database according to the test version. The user will not see the page flickering.

* What happens if the page is cached? The part you're testing could be fragment-cached or the whole HTML of the page could be cached or it could be cached even with CDN. How it will affect your setup?

Solutions:
1. Load two HTML versions in the frontend and display dynamically with javascript (the test version is stored in the cookie) --> The logic for either displaying Version A or B lives in the javascript in the frontend, and not in the Ruby backend (erb)
2. Use HTTP header cache:private --> cacheing is done only locally in the browser and not any middleware like CDN

* How you'd abstract your categorization logic? If you have many tests running at the same time you'd duplicate quite a lot of code, I think it's a very interesting problem to think about.
