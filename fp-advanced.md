# Functional Programming advanced
## Algebraic data types
In functional languages such as Haskell. We have the ability to define algrebraic data types.
They look as follows:
```hs
data Fruit = Apple | Orange
```
the name on the left `Fruit` is called a *data type*. The things on the right seperated by `|` are called *data constructors*.
We can have a value of type `Fruit` and it can be either an `Apple` or an `Orange`.
This gives us the ability to pattern match against the data constructors. Like so:
```hs
tellMeWhichFruit :: Fruit -> String
tellMeWhichFruit Apple = "this is an apple"
tellMeWhichFruit Orange = "this is an orange"
```

In JavaScript world, we don't have a first class support for ADTs and so we need to use some kind of library to take advantage of some predefined ATDs, such as Maybe. If you wish to use Maybe, consider using the SancturaryJS library.

Let's now have a look how one would use Sanctuary in JS

By the way Maybe is defined like this:
```hs
data Maybe a = Nothing | Just a
```
it is a parametrized ADT and can carry a value.
```js
const maybeNumber = Just(5) // the type of maybeNumber is Maybe Number
```
So we have maybe a number, but what if we need to apply some kind of function to that value. Let's say we need to add 3 to it. This is where `map` comes in, as it is a common abstraction to apply a function to a value inside.
```js
const maybeNumber = Just(5)
const maybeBiggerNumber = maybeNumber.map(x => x + 3) // Just(8)
```
Now this alone isn't very usefull just yet, we could add numbers together long before we knew Maybe existed. How about this then.

The interesing property is this:
```js
const maybeNumber = Nothing
const maybeBiggerNumber = maybeNumber.map(x => x + 3) // Nothing
```
We applied a function to empty value and our app didn't crash, it gracefully returned Nothing.
Let's have a look at a less trivial example.
```js
const head = arr => arr.length > 0 ? Just(arr[0]) : Nothing
const nestedArray = [[1, 2, 3], [4, 5, 6]]

head(nestedArray) // => Just([1, 2, 3])

head(nestedArray).map(head) // => Just((Just(1)))
```
Oh well, we now how to many Maybes! And for that, we have our friend chain, that doesn't wrap the result back.
```js
head(nestedArray).chain(head) // => Just(1)
```
This way, we can safely handle otherwise unsafe values such as `null` or `undefined`.
Maybe this all looked very weird to you. Well let's try having a look at more familiar example. `chain` is sometimes also called `flatMap`
```js
const makePair = num => [num, num]
makePair(1) // => [1, 1]

[1,2,3].map(makePair) // => [[1, 1], [2, 2], [3, 3]]
[1,2,3].flatMap(makePair) // => [1, 1, 2, 2, 3, 3]
```
As mentioned before. It is all just a common API to deal with the same operations.

So Maybe is how you handle failure, but guess what, Maybe has a brother, and he is called Either. Either is a lot like Maybe, except its "Nothing" has a parameter so it let's you carry some sort of error message.

## Type classes
Let's have a look at the most common type classes and what defines them.
You can look at these as kinds of interfaces that you know from different languages. Each typeclass requires the type to implement some methods.
There's a bunch of well known abstract type classes based on Category Theory.
The definitions you can find about them can therefore be very abstract and very hard to digest. The scary names are used, because not only does it give us a common API, it also guarantees that some laws apply and that can give us a lot of stuff for free in return.

### The overview
#### Functor
from documentation: The Functor class is used for types that can be mapped over.
In layman terms, the type must implement the `map` method.
So we already know an array is a `Functor`, because it has the `map` methods.

#### Semigroup
The type must implement the `append` function.

#### Monoid
A Semigroup with identity element.

#### Monad
Must implement the `chain` and `of` function.

#### Applicative
Must implement the `ap` function.

### The laws
The implementation of these methods must obey laws, such that identity is preserved for `map`.
e.g.
```
map id == id
map (f . g)  ==  fmap f . fmap g
```
where `id` is the identity function
That doesn't really have to bother us at this point.

This was just to put everything in context. It's not as scary as the words make it look. It's just some common ground so we don't keep reinventing the wheel over and over. The foundation on which it stands is very abstract, and so the definitions tend to be very hard to digest. Worry not, though, we'll look at examples.

## The Maybe monad
It is called a monad, because it implement several of the aforementioned type classes, from which, `monad` has the strongest implications.
But yeah, you were using monads all this time (Arrays) and you didn't even know. But wouldn't it be nice if everything had similar API?
There is a very familiar story with a sad end in the JavaScript world.

## The continuation monad story
When Promise spec was formed, it had the perfect opportunity to follow monadic laws, it was suggested here
https://github.com/promises-aplus/promises-spec/issues/94
But was dismissed and the Author of such proposition was told to go back to his Fantasy Land, and he responded, by creating a JavaScript specification called FantasyLand (https://github.com/fantasyland/fantasy-land) that defines ADTs that follow these laws.
There are alternative libraries replacing Promises, to allow to take advantage of these common APIs.
