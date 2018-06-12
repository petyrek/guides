# Functional programming
Defining your program in terms of pure composable, reusable functions.

## Why?
- pure functions are easy to test - they are predictable and predictability leads to less bugs
- less magic is happening, everything is more explicit, so you can see what are all the inputs in the consideration, unlike OOP, where you have to chase mutable state across multitude of files, human brain just cannot efficiently do that

## How can you do something usefull without side effects?
You cannot. The user, the network, the database, all of which we have to deal with *are impure*.
The key idea is to limit the impure surface as much as possible. Define as much as you can in a pure fashion.

## FPers use fancy words like monads, it is just too difficult!
That's just an unfortunate misconceptions. The underlying concepts are rather straight forward. We'll have a look at concrete examples which should help shed light on some of this.

## Function signatures
We first need to understand the notation of function signatures used in funcational languages. When someone says pure FP, Haskell comes to mind to most people. The language that rather strongly represents the FP ideas.
For this reason, Haskell function signatures are common in the FP world. Let's dissect it.

Let's demonstrate on the curried add function
```js
// add :: Num a => a -> a -> a
const add = x => y => x + y
```
So what does the signature tell us?
Let's first ignore the part before the fat arrow (`=>`)
```js
a -> a -> a
```
tells us, that this function accepts two arguments of type `a` and returns a value of type `a`. So all the inputs and the outpus have the same polymorphic type `a` and can be any type. There are no parentheses because the arguments are taken one by one, so you can apply how many of the you'd like and you'll get the corresponding result.

However, the part before the fat arrow symbolises a typeclass constraint. `Num a` means, that `a` can be of any type, that supports the `Num` typeclass, which is the typeclass that supports the `+` operator amonst others.
In JavaScript, there is only a single data type that supports `+` and that is `number` so we can specialize the type signature to this
```js
add :: Number -> Number -> Number
```

## Different type classes
There's a lot of different usefull type classes. Names of which are feared by many. Let's name a few.
- Semigroup
- Monoid
- Functor
- Applicative
- Monad

They all serve a very simple purpose. To unify the API of working with different data types.
Instead of coming up with a different name to add two values of `CustomerBalance` together, you implement the Semigroup type class, which then allows you to add them together using the `append` function, or the `<>` operator, or what have you in your particular language.

If you have an array of `CustomerBalance` values and want to sum them using reduce, you implement the Monoid typeclass. Monoid is just a Semigroup that has a neutral element. For `CustomerBalance` it would probably be `0`.

## Folds
So perhaps you have heard of this weird word `fold`, in relation with foldable data structures. In JavaScript world there is an equivalent of `foldLeft` called `reduce`.

Let's see how we could define a function that sums an array.
```js
const sum = arr => {
  let tmp = 0
  for(let i = 0; i < arr.length; i++) {
    tmp += arr[i]
  }
  
  return tmp
}
```
Let's see how we could define a function that calculates the product of an array.
```js
const product = arr => {
  let tmp = 1
  for(let i = 0; i < arr.length; i++) {
    tmp *= arr[i]
  }
  
  return tmp
}
```
This is a lot of boilerplate and most of the code is the same. What is different is the operator and the initial value. Turns out, we can refactor this in terms of folds.
```js
const sum = arr => Ramda.reduce((x, acc) => x + acc, 0, arr)
const sum = arr => Ramda.reduce((x, acc) => x * acc, 1, arr)
```

So let's have a look at the type signature of `reduce`.
```js
Foldable t => (b -> a -> b) -> b -> t a -> b
```
So we can use reduce with any foldable data type, which, in case of JavaScript is only an array, so we can specialize.
```js
(b -> a -> b) -> b -> [a] -> b
```

So folds are nothing else, than an abstraction over loops (in JavaScript).
