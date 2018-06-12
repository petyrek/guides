# Functional programming basics
So as functional programming is growing in popularity, you may have heard a thing about it.
Here we will introduce the basics.

Let's first enhance our vocabulary.

## Pure function
Is a function that given the same input, always produces the same output.

## Immutable data structure
Is a structure, the values of which, you cannot change.
If you perform any operation on such structure, instead of modyfying the existing, it produces a new structure, with the values changes.

This isn't the default in JavaScript and therefore requires discipline to adhere.
An example of this could be the `Array.prototype.sort` function. It mutates the original array, so after you sort it, your original array is not the same anymore. This could lead to unexpected and hard to trace bugs.

A way out of this, is to use some kind of functional utility library, such as [Ramda](https://ramdajs.com/)

## Currying
In most functional languages, functions are curried. That means that they take their argument one by one.
Again, this isn't the default in JavaScript and in order to achieve it, we have to define our functions in a slightly different way.
```js
const add = a => b => a + 5

add(5)(3) // => 8
```
Now what good does this do us?
We can use it to create something called a *partially applied function*
```js
add(5) // Function
```
Now we got ourselves a function back, that is still waiting for the remaining arguments.
We can use such constructs to create a lot of *small*, *reusable* functions.
```js
const increment = add(1)

increment(5) // => 6
```
We will see later on how this can be used to write compact, expressive code.

## Higher order functions
A HOF is a function, that accepts a function as its argument. There are 3 functions reasonably well adopted among JavaScript developers. These are `map`, 'filter' and `reduce`. The last one is usually the head scratcher for most people. We'll look at that one in the more advanced article.

Let's have a look at `map` (`Array.prototype.map`).
```js
[1, 2, 3, 4].map(x => x + 1) // => [2, 3, 4, 5]
```
`map` takes a function and applies that function to every element of the array and returns a new array.
But we could use our partially applied function `increment` from above. We don't need to be creating an intermediate arrow function.
```js
[1, 2, 3, 4].map(increment) // => [2, 3, 4, 5]
```
