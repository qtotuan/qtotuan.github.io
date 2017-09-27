---
layout: post
title:  "What is React?"
date:   2017-07-30 12:11:03 +0200
categories: react
---
How often have I heard, "We are coding it like this right now, but once you learn React..."

It always seemed like the moment that we learn React would be a turning point. Like an epiphany, everything will fall into place, my life will finally make sense!


## What the heck is React?

"React is a declarative, efficient, and flexible JavaScript library for building user interfaces."  
(https://facebook.github.io)

OK.

So what I get from that is:

  1. It is a library. So it is not a web framework like Rails.
  2. It is advertised as "flexible" - how so?

Upon further research here are my more detailed answers to the beginner's React mind:

You know how we have the MVC (Model-View-Controller) framework in Rails? React is the "V" - the view, the frontend. With React I can make web components that I can reuse over and over in my app and even across multiple apps. This makes the code more maintainable, because it is organized - something that becomes important the bigger and more complicated the app is. It is like meal prep - once the items are prepared, you just need to assemble the parts and mix'n match.

The components don't have any data! They know HOW to be rendered on the screen but not WHAT. Whenever the data changes, the components will update.


## Why do so many people love React?

The reason why developers use React is that the DOM is slow and React speeds up the process. Every time something changes, the browser has to recalculate the CSS, do the layout and re-render the whole page. This takes time. React does not manipulate the DOM itself but it makes a copy it, a "virtual DOM" that it updates and then renders.

The process goes something like this:
  * Something changes and the entire UI will be copied into a virtual DOM
  * React calculates the difference between the old version and the new version of the virtual DOM
  * React updates the real DOM with ONLY what has changed, not the entire UI


## What are Web Components?

They are HTML templates that are reusable and can be imported into different files. They can be used as HTML tags. For example, if you created a component called "MyFirstComponent", you could use it as such:

```
import React from 'react'
import ReactDOM from 'react-dom'

const MyFirstComponent = ({name}) => {
  return `Hello, ${name}!`
}

class MyApp extends React.Component {
  render () {
    return <MyFirstComponent name="Diane" />
  }
}

ReactDOM.render(<MyApp />, document.getElementById('root'))
```

Looks just like a normal HTML tag right?


## But why Web Components?

The advantages of web components are:

  1. Design with reusable components stays consistent - you can use your web components in multiple places with different data
  2. Your code stays organized and is thus more maintainable
  3. You are practically building out your own library. Your components will be properly name spaced and functionalities will not leak into each other. The larger your projects become, the more evident the advantage of this type of organization.


Maybe you have noticed from the example above that we use "const" to create MyFirstComponent and "class" for App. Both are web components. The two types are called Container Components and Presentational Components. In our example above, "App" is a container component and "MyFirstComponent" is a presentational component.

"Container Components:
Hold the logic of the component - how things WORK (behavior)
Provide the data to presentational components
Have a state, as they can serve as data sources

Presentational Components:
Hold the view of the component - how things LOOK
Have no dependencies on the rest of the app
Receive data and callbacks from a parent component
=> Some call this type of component 'dumb', vs. container components that are 'smart'


The advantage of architecturing your app this way is that you are following the principle of separation of concerns. You are forced to single out pieces of code and contain their functionality. It helps you to understand your UI and app better. And imagine that you are working with a designer. You can just have them tweak your components' design to their hearts' desire without you having to worry that they break anything in your app's logic.


So,  yeah. "Once you learn React..." - the truth is slowly revealing itself to me now...
