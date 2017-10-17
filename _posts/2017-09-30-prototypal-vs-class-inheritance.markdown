---
layout: post
title:  "Object Composition over Class Inheritance"
date:   2017-09-30 12:11:03 +0200
categories: javascript
---
During my prep work for my job hunt I came across an interesting interview question: ["What is the difference between Prototypal Inheritance and Class Inheritance?"](https://medium.com/javascript-scene/master-the-javascript-interview-what-s-the-difference-between-class-prototypal-inheritance-e4cd0a7562e9)

It is a loaded question. There are different opinions on the benefits and best practices on using them. And to come up with a decent answer, one has to fundamentally understand JavaScript, its [protoype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain), and [different types of prototypal inheritance](https://medium.com/javascript-scene/3-different-kinds-of-prototypal-inheritance-es6-edition-32d777fa16c9).

Spoiler Alert - the below dogma from the influential book ["Design Patterns" by the Gang of Four](https://en.wikipedia.org/wiki/Design_Patterns) has been referenced in countless blogs and talks.
> Favor object composition over class inheritance.

What is the difference between these two?

Object Composition means creating new instances of objects not using classes but by compiling object prototypes.

> Classes inherit from classes, thus creating a hierarchy of subclasses

Let's look at examples to makes this more clear. It seems like tradition to explain inheritance using animals, so here we go.

For example, cats and dogs are both animals. So we can say that "Animal" is the parent class of "Cat" and "Dog". Cats and dogs have some things in common but differ in their specific behavior. Look at this example:

```
Animal
.breathe
.move

  Cat
  .meow

  Dog
  .bark
```

Now, let's add a jellyfish:
```
Animal
.breathe
.move

  Cat
  .meow

  Dog
  .bark

  Jellyfish
  .sting
```

See the problem? A jellyfish is an animal that moves, but it does not breathe. But because of __class inheritance__ "Jellyfish" gets all the methods from "Animal", even though it does not need it.

What we could do is to move "breathe" down to a lower hierarchy level, and define it twice in "Cat" and "Dog", like this:

```
Animal
.move

  Cat
  .breathe
  .meow

  Dog
  .breathe
  .bark

  Jellyfish
  .sting
```
This would work, but we are duplicating code and violating the DRY principle.

And by now we can understand how class inheritance can create trouble: imagine that you are adding more subclasses, which demand you to rewrite the base class. This modification will have a rippling effect through all of the subclasses, not only the one you intended it for.

There must be a better way. Meet:

__Object Composition__

This allows us to mix and match precisely. Watch.

``` javascript
const breather = { breathe: () => console.log("... phew ...") }
const mover = { move: () => console.log("One, two, step") }
const purrer = { purr: () => console.log("rrrrrrrr") }
const barker = { bark: () => console.log("Woof!") }
const stinger = { sting: () => console.log("Bzz!") }

const createCat = (options) => {
  return Object.assign({}, breather, mover, purrer, options)
}

const createDog = (options) => {
  return Object.assign({}, breather, mover, barker, options)
}

const createJellyfish = (options) => {
  return Object.assign({}, mover, stinger, options)
}

createDog().move() // "One, two, step"
createCat().breathe() // "... phew ..."
createJellyfish().sting() // "Bzz!"
```

The example above shows that for each object we can choose exactly which properties it should get. No superfluous props that it does not need.

What makes Object Composition possible is Object.assign(). An empty object is combined with the properties of other objects, and merges into one. To me more exact, the properties of the source objects are copied to the new object.

So remember,
> Favor object composition over class inheritance.

<br><br>

Further reads:  
* [Common Misconceptions About Inheritance in JavaScript](https://medium.com/javascript-scene/common-misconceptions-about-inheritance-in-javascript-d5d9bab29b0a)  
* [Classless Javascript (Composition over Inheritance)](https://medium.com/front-end-hacking/classless-javascript-composition-over-inheritance-6b27c35893b1)  
* [Prototypal Inheritance in JavaScript](https://medium.com/@kevincennis/prototypal-inheritance-781bccc97edb)  
* [Composition Over Class Inheritance](https://www.youtube.com/watch?time_continue=272&v=wfMtDGfHWpA)
