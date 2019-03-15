# Asynchronous evolution with Node.js

1. Syncronous programming
2. Asyncronous programming
3. Callback
4. Async.js
5. Promise
6. Generators
7. Async/await

## 1. Syncronous programming

In traditional programming practice, most I/O operations happen synchronously. If you want to read a file, the main thread will be blocked until the file is read, which means that nothing else can be done in the meantime. To solve this problem and utilize your CPU better, you would have to manage threads manually.

Node.js is single-threaded - from a developer's point of view.

In synchronous programming with Node.js, the execution of additional JavaScript in the Node.js process is blocked until a non-JavaScript operation completes, meaning it can do absolutely nothing while the file is being read.

```javascript
var fs = require("fs");

var data = fs.readFileSync('input.txt');

if (data) {
    console.log(data.toString());
}
console.log("End");
```

The first example shows that the program blocks until it reads the file, and then only it proceeds to end the program.

### Error handling

A standard practice to handle synchronous errors is to throw and catch the error. When you throw an error, it becomes an exception.

The instruction *throw* delivers an error synchronously — that is, in the same context where the function was called. If the caller (or the caller's caller, ...) used try/catch, then they can catch the error. If none of the callers did, the program usually crashes.

Here's an example of using an error as an exception:

```javascript
throw new Error('something bad happened');
```

And here is an example of catching an error:

```javascript
var fs = require("fs");

try {
  var data = fs.readFileSync('invalid.txt');
  console.log(data.toString());
} catch (ex) {
  console.error(ex);
}

console.log("End");
```

# 2. Asynchronous programming

Node.js introduced an asynchronous programming model. Asynchronous I/O is a form of input/output processing that permits other processing to continue before the transmission has finished.

With asynchronous programming in Node.js, a function may start non-JavaScript operation operation and return the control to the execution environment immediately so that the next instruction can be executed.

This makes Node.js highly scalable, as it can process a high number of requests without waiting for any function to return results.

## 3. Callbacks

Callback is a function provided specifically for handling errors and the results of asynchronous operations. The user passes you a function (the callback), and you invoke it sometime later when the asynchronous operation completes.

Node makes heavy use of callbacks. All of the I/O methods in the Node.js standard library provide asynchronous versions, and accept callback functions.

```javascript
var fs = require("fs");

fs.readFile('input.txt', function callback(error, data) {
   if (error) return console.error(error);
   console.log(data.toString());
});

console.log("End");
```

This example shows that the program does not wait for file reading and proceeds to print "End" and, at the same time, the program without blocking continues reading the file.

This is possible beacuse of the "Event Loop", a mechanism of Node.js for scheduling asynchronous operations.

See "The Node.js Event Loop" documentation
https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/

### Error handling

Error handling is a pain, and it's easy to get by for a long time in Node.js without dealing with errors correctly. However, building robust Node.js applications requires dealing with errors properly, and it's not hard to learn how.

Callbacks are the most basic way of delivering an error asynchronously. The usual pattern is that the callback is invoked as *callback(error, result)*, where only one of err and result is non-null, depending on whether the operation succeeded or failed.

Having the first argument be the error is a simple convention that encourages you to remember to handle your errors. If *null* was not the first argument passed on success, the user would need to check the object being returned and determine themselves whether or not the object constituted an error - a much more complex and less user-friendly approach.

```javascript
var fs = require("fs");

var callback = function(error, data) {
   if (error) {
     return console.error(error);
   }
   console.log(data.toString());
}

fs.readFile('invalid.txt', callback);
```

An error is any instance of the Error class.

```
{ Error: ENOENT: no such file or directory, open 'invalid.txt'
    at Error (native)
  errno: -2,
  code: 'ENOENT',
  syscall: 'open',
  path: 'invalid.txt' }
```

Errors may be constructed and then passed directly to another function.

```javascript
callback(new Error('something bad happened'));
```

All of your errors should either use the Error class or a subclass of it. You should provide name and message properties.

The name property is used to programmaticaly distinguish between broad types of errors (e.g., illegal argument, incorrect type, connection failed). The message property must provide a human-readable error message.

### Callback hell

Asynchronous JavaScript, or JavaScript that uses callbacks, is hard to get right intuitively. A lot of code ends up looking like this:

```javascript
api.get('/auth/:username', function(request, response) {
    let username = request.params['username']
    let password = request.params['password']

    database.getConnection(function(err, connection) {
        if (err) {
            response.status = 500;
            return response.send('Internal Server Error')
        }
        connection.selectUser(username, function(err, user) {
            if (err) {
                response.status = 500;
                return response.send('Internal Server Error')
            }
            else if (!user) {
                response.status = 404;
                return response.send('Not Found')                
            }
            else if (user.password === password) {
                connection.selectPermissions(user.username, function(err, permissions) {
                    if (err) {
                        response.status = 500;
                        return response.send('Internal Server Error')
                    }
                    else if (permissions.administration === true) {
                        user.accesses =  user.accesses + 1
                        connection.saveUser(user, function(err, user) {
                            if (err) {
                                response.status = 500;
                                return response.send('Internal Server Error')
                            }
                            reponse.redirect('/home');
                        });
                    }
                    else {
                        response.status = 403
                        return response.send('Forbidden')
                    }
                });
            }
            else {
                response.status = 401
                return response.send('Unauthorized')
            }
        });
    });

    // to sign a token
    // https://www.npmjs.com/package/jsonwebtoken

    // @TODO create a sample app with Koa
});
```

You can avoid the callback hell with modularization.

```javascript
function authorize(connection, user, permissions, response) {
    if (permissions.administration === true) {
        user.accesses =  user.accesses + 1
        connection.saveUser(user, (err, user) => {
            if (err) {
                response.status = 500;
                return response.send('Internal Server Error')
            }
            reponse.redirect('/home');
        });
    }
}

function authenticate(connection, user, password, response) {
    if (user.password === password) {
        connection.selectPermissions(user.username, (err, permissions) => {
            if (err) {
                response.status = 500;
                return response.send('Internal Server Error')
            }
            authorize(connection, user, permissions, response);
        });
    }
    else {
        response.status = 401
        return response.send('Unauthorized')
    }
}

function login(connection, username, password, response) {
    connection.selectUser(username, (err, user) => {
        if (err) {
            response.status = 500;
            return response.send('Internal Server Error')
        }
        else if (!user) {
            response.status = 404;
            return response.send('Not Found')            
        }
        else {
            authenticate(connection, user, password, response);
        }
    });
}

function auth(username, password, response) {
    database.getConnection((err, connection) => {
        if (err) {
            response.status = 500;
            return response.send('Internal Server Error')
        }
        login(connection, username, password, response)
    });
}

api.get('/auth/:username', function(request, response) {
    let username = request.params['username']
    let password = request.params['password']

    auth(username, password, response);
});
```

## 4. Async.js

Sometimes you need to handle asynchronous operations in a control flow.

A tool you can use to handle these kind of issues is Async.

Async is a utility module which provides useful functions for working with asynchronous JavaScript. It helps to structure your applications and makes control flow easier.


```javascript

function auth(database, username, password, response, callback) {
    database.getConnection((err, connection) => {
        if (err) {
            response.status = 500;
            return response.send('Internal Server Error')
        }
        login(connection, username, password, response)
    });
}

api.get('/auth/:username', function(request, response) {
    let username = request.params['username']
    let password = request.params['password']

    async.waterfall([
        function(callback) {
            callback(null, database,username, password, response)
        },
        auth,
        login,
        authenticate,
        authorize
    ])

    async.waterfall([auth, login, authenticate, authorize])
    auth(username, password, response);
});
```





## 5. Promises

The Promise object is used for deferred and asynchronous computations. A Promise represents an operation that hasn't completed yet but is expected in the future.

### Error handling

Promise rejections are a common way to deliver an error asynchrously. This method is growing in popularity since the release of Node.js version 8 that includes support for async/await. This allows asynchrounous code to be written to look like synchronous code and to catch errors using try/catch.

...

## 4. Generators

...

## 5. Async/await

...


# References

[Tutorials Point - Node.js - Callbacks Concept](https://www.tutorialspoint.com/nodejs/nodejs_callbacks_concept.htm)

[Understanding Async Programming in Node.js](https://blog.risingstack.com/node-hero-async-programming-in-node-js/)

[Callback Hell](http://callbackhell.com/)

[What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

[Overview of Blocking vs Non-Blocking](https://nodejs.org/en/docs/guides/blocking-vs-non-blocking/)

[What are the error conventions?](https://docs.nodejitsu.com/articles/errors/what-are-the-error-conventions/)

[What is “callback hell” and how and why RX solves it?](https://stackoverflow.com/questions/25098066/what-is-callback-hell-and-how-and-why-rx-solves-it)

[Error Handling in Node.js](https://www.joyent.com/node-js/production/design/errors)

[REST Countries](https://restcountries.eu/)

[Managing Node.js Callback Hell with Promises, Generators and Other Approaches](https://strongloop.com/strongblog/node-js-callback-hell-promises-generators/)

[Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)

[Node.js Async Best Practices & Avoiding the Callback Hell](https://blog.risingstack.com/node-js-async-best-practices-avoiding-callback-hell-node-js-at-scale/)

[Generators in Node.js: Common Misconceptions and Three Good Use Cases](Generators have been all the rage lately. Many Node developers (including myself!) are excited and intrigued about writing their asynchronous code like this:)

...

