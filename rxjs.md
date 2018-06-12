# RxJS

## Key concepts
- *Observable* - stream of future values
- *Observer* - what we use to get the values out of the observable
- *Operators* - methods on the Observable object that return new, altered stream


## So how can an `Observable` look like?
 In order to look at one, we first have to create one. So first order of business, we look at creating an `Observable` using a **creation** operators.
```js
Rx.Observable.fromEvent(button, 'click')
```
```js
Rx.Observable.of(1,2,3)
```
```js
Rx.Observable.fromPromise(fetch('http://swapi.co/api/people'))
```

Okay, so we have an observable, what do we do with it?

Well that's where the `Observer` comes in.

## What does an `Observer` look like?
```js
const observer = {
  next: () => {}
  error: () => {}
  complete: () => {}
}
```

Yeah, it's just and object with three callbacks. Simple like that.

Let's implement our own observable and try to make a better sense of all this.

```js
Rx.Observable.create(observer => {
  observer.next(5)
  observer.next(10)
  observer.complete()
})
```
```js
Rx.Observable.create(observer => {
  let i = 2;
  setInterval(() => {
    observer.next(i)
    i = i ** 2
  }, 100)

  setTimeout(() => observer.complete(), 350)
})
```

## Graphical representation of an Observable = marble diagrams
let's see a marble diagram for the Observable we created ourselves

`--2--4--8-|`

## And finally, what about them operators?

Operator is either a method on the Observable stream, such as
```js
Rx.Observable.of(1,2,3)
  .map(x => x * 2)
```

or a static method on the `Observable` class e.g. the creation operator
```js
Rx.Observable.of(1,2,3)
```

## Let's reherse the graphical representation when operators are involved

```js
--2--4--6--8|
map(x => x * 2)
--4--8--12-16|
```
## Let's make sure we understand what an operator is, let's implement our own filter operator
```js
Rx.Observable.prototype.myFilter = function(predicate) {
  return Rx.Observable.create(observer => {
    this.subscribe({
      next: (v) => predicate(v) && observer.next(v),
      error: (err) => observer.error(err),
      complete: () => observer.complete()
    });
  });
}
```

And there you have it!

```js
--2--4--6--8|
myFilter(x => x < 5)
--2--4------|
```

So what it really comes down to is operators

Now let's suppose we have the following code
```js
const apiGet = url => new Promise(resolve =>
  setTimeout(() => resolve(url.split('?')[1]), 1000)
)


Rx.Observable.interval(1000)
  .map(x => `www.url.com?itemId=${x}`)
  .map(x => apiGet(x))
  .subscribe(
    console.log,
    console.error,
    console.warn
  )
```

Oh noes! We're getting back
```js
[object Promise] { ... }
```

That's not what we wanted!
What we got actually, is a `Higher order observable`.
An `observable` wrapped inside another `observable`.

Now there are several operators that can help us out with that. Namely:
- switchMap
- mergeMap
- concatMap

The difference in these is how subsequential observables emitted are merged.

Other operators worth mentioning
- Combination operators - for handling multiple observables as one
  - race
  - concat

## Okay so that was RxJS, now why bother? Don't we have enough ways to handle async?
Provides a declarative way how to handle **multiple** values asynchronously and is `PUSH`ed to the consumer. This means that the producer determines when data is pushed to the consumer and the consumer simply *reacts* to it.

Generators on the other hand, not only are they weird looking and filthy imperative, you have to `PULL` the values manually, so the consumer decides when data is delivered.

## Gimme a real usecase
Well for starters, as I've talked about previously, you could start by throwing `redux-saga` out the window and start using `redux-observable`.

If we took it a step further we could throw out the whole `Redux` ecosystem and replace it with `MobX`. Though `MobX` isn't explicit about the usage of `Observables` and is more magic based. Which does reward you by being able to handle everything you could ask for (such as async) out of the box and less boilerplate. Though for the price of not enforcing immutability, purity and is less explicit in what happens with the app state.

Other alternatives are not as mature, such as `CycleJS`.
Despite RxJS being around for quite a while, it's still early days for RxJS as the community has just started picking deeper interest in it.
