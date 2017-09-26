---
layout: post
title:  "Chocolate Factories Produce Chocolate. Factory Functions Produce... what now?"
date:   2017-09-21 12:11:03 +0200
categories: javascript
---
What is a factory function? It is a function that returns an object literal. Actually any function in JavaScript can return an object literal. When it is not a class is it called a factory function.

To understand factory functions it is helpful to understand how they are different from classes and their constructor functions.

Check out this example below.

``` javascript

class Dog {
  constructor(name) {
    this.name = name,
    this.sound = "woof!"
  }

  talk() {
    console.log(this.sound)
  }
}

let fido = new Dog("Fido")
fido.talk() // => "woof!"

```

What we are doing is creating a Dog class, which has a property "sound", and a method "talk" that accesses the "sound".

What happens if we try to pass the method "talk" to another function outside of the class?

``` javascript

$('.button').click(fido.talk)

```
This will not achieve the desired result because the keyword "this" from "this.sound" in the class is now referring to the DOM element, not the Dog instance!

We could solve it with bind():

``` javascript

$('.button').click(fido.talk.bind(fido))

```

Would we get this same problem if we used a factory function? Let's try it out:

``` javascript

function dog(name) {
  let sound = "woof!"
  return {
    name: name,
    talk: () => console.log(sound)
  }
}

let fido = dog("Fido")
fido.talk() // => "woof!"

$('.button').click(fido.talk)

```
Now "dog" is just a function which returns an object that has the property of "talk". It has a value of a function which has access to the variable "sound". Remember closures? Because of the closure, "talk" remembers the environment where it has been created. This means that we can call "talk" outside of the function where it has been created and still have access to "sound".

So now, we can actually pass the "talk" method without any problem to our click handler and it works:

``` javascript

$('.button').click(fido.talk)

```
***********

Side Note

There is difference when you try to access sound from a class instance vs a factory made object. With classes, we can access it like this:

``` javascript

class Dog {
constructor(name) {
  this.name = name,
  this.sound = "woof!"
}

talk() {
  console.log(this.sound)
}
}

let fido = new Dog("Fido")
fido.sound // => "woof!"

```

Accessing "sound" directly from outside the "dog" function is not possible - the variable is private to the "dog" function.

***********

When should you even consider using classes over factory functions?

Honestly this question is hard to answer and there are many discussions. Check out [this one][eric-elliott] for example.

A good answer might be: "It depends."

One benefit of factory functions over classes is the syntax. Classes require the keyword "new", which developers might forget. Another problem that factory functions do not have is deep inheritance hierarchies with 'extends'.

Classes can be favorable when you are coming from a class based language, like Ruby or C#, where you are taught to use classes and constructors. Also, creating a class object is a little bit faster than creating an object via a factory function. So if your website is creating thousands of object per frame, there is a slight performance difference.



(Many thanks to and heavily inspired by [Fun Fun Function][fun-fun-function])


[eric-elliott]: (https://medium.com/javascript-scene/javascript-factory-functions-vs-constructor-functions-vs-classes-2f22ceddf33e)
[fun-fun-function]: https://www.youtube.com/watch?v=ImwrezYhw4w
