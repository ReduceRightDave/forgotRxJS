Haven't used RxJS for a while? These terse and compact notes might jog the memory.

RxJS - Observables
==================

* RxJS is the JavaScript implementation of Observables

* Observables are like promises, but can produce more than one value. They're both ways to avoid callback hell, ie handle asynchronous events more sanely

* Observables are returned from things like Angular's `httpClient.get`

* If you create an observable object with `new Observable(producer)`, you pass it a producer function. Much like the function you pass to `new Promise()`, this producer function potentially does async stuff

* For objects of type `Observable`, many developers use a variable name ending in `$`
 <!-- https://medium.com/@benlesh/observables-and-finnish-notation-df8356ed1c9b -->

* Observables can be created with RxJS static methods like `of`, `from`, `fromEvent`, `interval`. Eg creating an Observable from an array:
```javascript
import { from } from 'rxjs'
const iceCreams$ = from(['chocolate', 'vanilla'])
```

* When you call `subscribe(next)` on an observable, you pass it a callback -a so called *next* function. Example: 
```javascript
iceCreams$.subscribe(console.log) 
//console.log is the next function, and is invoked for each flavour
```

* When you call `subscribe(next)`, the producer function is invoked. The producer -where async stuff can happen -calls your next function, zero or many times, and can pass it values. This is the observable *emitting values* 

* Alternatively, call `subscribe(observer)` -observer being just a plain JS object with the keys `next`, `error` and `complete`, and a callback for each

* Some observables emit the complete event (invoking your complete callback), which means it's finished doing stuff and won't emit further values

* The key point is: the observable holds a reference to your observer, and any time the producer function feels like it, it invokes the next callback

* You can get an observable object from a promise by calling `from(promise)`


Cleaning up
-----------

* When you call `subscribe` you get back an object of type `Subscription`
```javascript
const subscription = iceCreams$.subscribe(console.log)
```

* This object has an `unsubscribe` method. This method stops the producer from doing any async stuff or emitting values

* If you don't at some point call `unsubscribe()`, memory leaks can happen

* In Angular, the `async` pipe in templates subscribes to an observable or promise and returns the latest value it emitted. When the component gets destroyed, the async pipe unsubscribes automatically!


Operators
---------

* Operators process values being emitted. Import them like so: `import { operatorName } from 'rxjs/operators'` 

* Observables have a `pipe` method, which allow you to apply operators. Rather than calling `subscribe(observer)` directly on your observable object, do 
```javascript
observable$.pipe(operator(args)).subscribe(observer)
```

* Operators can be chained, in the form `pipe(op1(args), op2(args))`. Each is applied each time the observable emits a value, one after the other

* Common operators are `map`, `filter` and `reduce` -similar to the array methods. But unlike their array counterparts, they are passed one value at a time

* With `map(callback)`, each value emitted from the source observable is processed in the callback and passed on 

* Example:
```javascript
import { from } from "rxjs";
import { map } from 'rxjs/operators';
const iceCreams$ = from(['chocolate', 'vanilla'])
iceCreams$.pipe(map((ic) => ic.toUpperCase())).subscribe(console.log)
//CHOCOLATE VANILLA
```

* With `filter(callback)`, each value emitted from the source observable is tested according to the callback, and possibly passed on

* `reduce(callback)` accumulates the values emitted from the source and 

* `scan(callback)` 

* `take(number)` 

* `tap(callback)`

* `takeUntil(observable)` 

* There are many more operators

<!--

Error handling
--------------
https://rxjs-dev.firebaseapp.com/api/operators/catchError

* Error handling also uses operators
```javascript
import { catchError } from 'rxjs/operators'
.pipe(
  catchError(errorHandlingFunction)
)
```
-->

How operators work (conceptually, at least)
-------------------------------------------

* In `observable$.pipe(operator1(args)).subscribe(next)`, the invocation of `operator1` returns a function that `pipe()` will invoke; this function is passed the first/source observable, and returns a new observable 

* The new observable is created with a producer function, which, for the most common operators, does not itself do any async stuff. Rather...

* The producer calls `subscribe` on the source observable, processes the values emitted, and invokes `next`, `error` and `complete` on this observable's observer, according to what this operator does

* This new observable is the object on which your `subscribe` is actually called. So a chain of subscriptions is set up

* For `pipe(operator1(), operator2())` a longer chain of subscriptions is set up

* It's important to distinguish between the time that this chain is set up, and the later point in time when events start to be emitted and observer next functions actually get invoked -what you might call *event time* 


Responding to an event by triggering more async
-----------------------------------------------

* Some operators expect to be passed a callback whose invocations create a new observable -an *inner observable*

* In response to an event being emitted by the source observable, they can call `subscribe` (and possibly `unsubscribe`) on this newly created inner observable -potentially triggering other async operations

* Common examples are `switchMap`, `mergeMap` and `concatMap`

* With `switchMap`, each time a new inner observable is created, unsubscribe is called on the previously created inner observable

<!-- * Example TODO -->



Subject
-------

* Subject is a type of Observable and allows you to emit values manually

* Emit values by calling the `next` method

* Subscribe to it as you would any other observable


* Subjects are multicast
