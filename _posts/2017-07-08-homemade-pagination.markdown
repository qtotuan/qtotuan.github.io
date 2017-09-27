---
layout: post
title:  "Homemade Pagination"
date:   2017-07-08 12:11:03 +0200
categories: javascript
---
This week we had an interesting mini-project to do: clone your favorite website and its search functionality. Make use of Ajax to make API calls and render the results without refreshing the page.

So off we go. And BAM! API was successful and the data was received!

But... then I wondered: "Should we really display the 1037 results at once?"

No, we should not! The user should see no more than 20 results per page and be able to click on buttons to navigate to the next/previous pages.

<br>
![flipping pages]({{ site.url }}/assets/170708_flipping_pages.gif){: .center-image }
<br><br>

Let's make some pagination! I would love to have something like this:

![pagination example]({{ site.url }}/assets/170708_pagination_example.png)
<br>

My ambition was to come up with my own pagination solution without using any packages and googling solutions. So I built my own paginator from scratch. This is how it goes:

The API that I am using is the food database Nutritionix, which holds nutritional values like calories, fat, protein, etc.

The API allows for specifying "offset" and "limit" in the queries. "Offset" is the starting point in the result set for paging and "limit" is the number of results that should be returned. I set the default for offset to "0", limit to 20 and pass in the values in the data key of the Ajax call:

``` javascript
function getItemData(searchTerm, callback, offset = 0) {
  $.ajax({
    url: 'https://api.nutritionix.com/v1_1/search/',
    type: 'post',
    data: {
      "appId": appId,
      "appKey": appKey,
      "query": searchTerm,
      "offset": offset,
      "limit": 20
    },
    success: function(data) {
      callback(data)
    }
  })
}
```

So, if the user clicks "Next" I want to show them the next 20 results. Two things need to happen: firstly, the currentPageNumber needs to be updated. Secondly, the new API call needs to pass in the offset incremented by 1. This is handled in the click listener of the button:

``` javascript
function showNextButton() {
	$("#pagination").append('<li class="page-item"><a class="page-link" href="#" id="next-button">Next</a></li>')
	$("#next-button").on("click", e => {
		e.preventDefault()
		currentPageNumber += 1
		getItemData(currentSearchTerm, showItems, 20 * (currentPageNumber - 1))
	})
}

function showPreviousButton() {
	$("#pagination").append('<li class="page-item"><a class="page-link" href="#" id="previous-button">Previous</a></li>')
	$("#previous-button").on("click", e => {
		e.preventDefault()
		currentPageNumber -= 1
		getItemData(currentSearchTerm, showItems, 20 * (currentPageNumber - 1))
	})
}
```


I will create the anchor tags that look like buttons programmatically. The button texts will start from the current page number (which is conveniently stored in the global variable) and will be incremented by 1 with each loop.  
I also need to pass in the data through the anchor tags so that the individual click listeners will be able to pass in the correct offset to the API call.  
And lastly, I need to check what the last page number is, otherwise the API call would error out.

``` javascript
function showPageNumberButton(total) {
	for (let i = 1; i <= 3; i++) {
		if (total - 20 * currentPageNumber <= 0) {
			return
		}

		let pageNumber = currentPageNumber + i
		$("#pagination").append('<li class="page-item"><a class="page-link" href="#" id="page${pageNumber}" data-page-number="${pageNumber}">${pageNumber}</a></li>')
		$('#page${pageNumber}').on("click", function(e) {
			e.preventDefault()
			currentPageNumber = $(this).data("pageNumber")
			getItemData(currentSearchTerm, showItems, 20 * (currentPageNumber - 1))
		})
	}
}
```

To tie it all together with some last logic:  
The "Previous" button only shows if the user is at least on page 2. The "Next" button only shows if there are more than 20 results for the whole query. The buttons with the page numbers only display if there are enough results to make up three more pages.

``` javascript
function showPagination(total) {
	$("#pagination").empty()
	if (currentPageNumber > 1) {
		showPreviousButton()
	}

	if (total > 80) {
		showPageNumberButton(total)
	}

	if (currentPageNumber * 20 < total) {
		showNextButton()
	}
}
```


My conclusion of building out this feature is that it feels like baking a cake for the first time.  
Is it edible? Yes.  
Is it delicious? It took much effort to make it, so hell yeah, it tastes delicious!  
Are there better recipes out there? For sure.

The point is that I made my "Triple Layered Pagination Cake" from scratch, having to come up with my own formula. It takes time and is at the same time very rewarding when the implementation is successful. We don't always have the luxury of time to build everything from scratch if you need to advance on a project. Additionally, we do not need to reinvent the wheel for every little detail - there must be hundreds of thousands of developers before me who have thought about pagination.  
But - it is nice to take one's time here and there to figure something out and code the heck out of a new feature.

Let me know your thoughts!
