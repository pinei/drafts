# Asynchronous Programming in Javascript

## What the Heck are Callback Functions?

Callbacks are one of the most used concepts of functional javascript.

Simply put, a callback function is a function passed into another function as an argument, which is then invoked inside the outer function to complete some kind of routine or action. _(Mozilla.org)_

```js
doSomethingWithCallback(function callback() {
  // ...
});
```

Callbacks are often used to continue code execution after an asynchronous operation has completed.

```js
setTimeout(function sayHello() {
  console.log("Hello");
}, 3000);
```

Now in a shorter syntax using ES6 lambda:

```js
var sayHello = () => console.log("Hello")
setTimeout(sayHello, 3000);
```

_Display a message after 3 seconds (3000 milliseconds)_

The 'setTimeout' function expects as its first argument a callback function to execute after a specified number of milliseconds.

A callback function can

Another practical example:

```
http.get("/service", function(data) {
  alert( "Data Loaded: " + data );
});
```

The 'get' function of an 'http' object expects a callback function that is executed if the HTTP request succeeds, with an additional payload of data resulted from the HTTP GET operation.

### TODO

- Convention about 'error' variable

## A Function with Callback

...

## Callback is a closure

When we pass a callback function as an argument to another function, the callback is executed at some point inside the containing function's body just as if the callback were defined in the containing function. This means the callback is a closure.

https://www.pluralsight.com/guides/javascript-callbacks-variable-scope-problem


## Promises 

Back in 2015, ES6 (ES2015) introduced the now very popular promises. A promise is an object that works like a contract, like an agreement that an asynchronous function will either resolve to a value or to an error. Promises are not the focus of this article, but they are really a foundation of the following topics, so make sure you understand them well first. There are excellent resources that explain promises in depth, one of my favorites is Mattias Petter Johansson’s Fun Fun Function video about Promises.

ES2015 also introduced Generator functions and the keyword yield. Generators can be used for both synchronous and asynchronous purposes, but it’s important to understand that they are the foundation on which the more popular async/await syntax is built on. Therefore, if you want to really understand async/await, you should first understand Generator functions, as well as the fact that they can be used for other use cases. Often tutorials start explaining generator functions by showing their usage equivalent to async/await, but this has some drawbacks. First, because generators functions can have other completely different uses which should be mentioned, and two, because it can make it more complicated to understand for someone who is not familiarized with them. Here I will start with the simplest example and then move on to their similarity with async/await.

The newer async/await syntax was initially proposed around 2015 for ES2017, to be able to write asynchronous code in a way that reads like synchronous code (without having to use .then() and callback functions). This is also described sometimes as “waiting code”. Async/await is a really useful way to create asynchronous code, which can be more compact and easier to understand than handling promises the regular way. However, they also have their limitations that are important to consider, which I’ll mention below. In some cases, it is still better to handle promises in the older fashion.

## Async / Await

```js
console.log('Starting...')

const LIST = ['Orange', 'Lemon', 'Banana' ]

process(LIST);

async function eachAsync(arr, fn) { // take an array and a function
   for(const item of arr) await fn(item);
}

async function process(list) {
    await eachAsync(list, async (data) => {
        let result = await print(data)
        console.log(result)
    });

    console.log('Done');
}

function print(data) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            console.log(data);
            resolve("Data was sent")
        }, 3000)
    });
}
```

### References

[Passing parameters to a callback function](https://stackoverflow.com/questions/3458553/javascript-passing-parameters-to-a-callback-function)

[Synchronous request in Node](https://stackoverflow.com/questions/8775262/synchronous-requests-in-node-js)