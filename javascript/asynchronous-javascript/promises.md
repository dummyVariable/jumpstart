# Promises

## What is a Promise?

> The `Promise` object represents the eventual completion \(or failure\) of an asynchronous operation, and its resulting value.

In its most basic terms, a promise is an object that defines a method called `then`. The promise object represents a value that may be available some time in the future. It greatly simplifies asynchronous logic in JavaScript.

### Promise vs Callback

Before we discuss Promise in more details, I would like you to see how it's used.

In the example below, we implement some logic to handle user logins to a server side application. In this login process, there are three asynchronous tasks, and each task depends on the previous task \(i.e. the output of one task becomes the input of the next task.\)

* A user login with user name and password.
* We search the database to get more information about this user.
* We update the user's profile information and save it back to the database.

![asynchronous tasks in a chain](../../.gitbook/assets/async-task.png)

If these tasks were synchronous tasks \(i.e. if they do not involve any asynchronous operations\), we could have written the codes like below:

```javascript
let user = User.login("user", "pass");
let results = query.find(user);
let result = results[0].save({ key: value });
// make use of the result
```

However, suppose the 3 tasks \(login/find/save\) are all asynchronous tasks, you cannot get the results right away after start executing a task. You have to provide some callbacks to process the results becomes ready.

The nested callbacks easily leads to the callback-hell approach:

```javascript
User.logIn("user", "pass", {
  success: function(user) {
    query.find({
      success: function(results) {
        results[0].save(
          { key: value },
          {
            success: function(result) {
              // the object was saved
            }
          }
        );
      }
    });
  }
});
```

Compare that with the following code with the much more elegant Promise workflow, with first-class error handling:

```javascript
    User.logIn('user', 'pass')
      .then(function (user) { return query.find(user); })
      .then(function (results) { return results[0].save({ key: value }); })
      .then(function (result) { // the object was saved })
      .catch(function (err) { // an error happened somewhere in the process });
```

Note: you still need to use callbacks with Promise. However, now the callbacks are not nested within each other anymore!!!

## ES6 Promise API

To fully understand the Promise API, let's take one example.

Let's say it is my wife's birthday this Friday. On Monday, I told her "I will have dinner with you on Friday". This is a **Promise**.

Before Friday, this promise is in **pending** state, and on Friday, it will be **resolved**.

At that time, there are two possible outcomes:

* I book a nice restaurant and have dinner with my wife. In that case, the promise is **fulfilled**.
* I got an urgent meeting on Friday evening, so I have to call my wife and tell her I couldn't have dinner with her. In this case, the promise becomes **rejected**.

Knowing that I don't always keep my promise \(based on my previous records\), my wife has the following plans:

* If the promise is **fulfilled**, she will buy me a gift in return.
* If the promise is **rejected**, she will let me sleep on the couch.

And she will observe the outcome of the promise and take actions accordingly.

One important property about the promise is that it can only be either fulfilled, or rejected. There is no 3rd outcome, and it cannot be both fulfilled and rejected at the same time.

Another important property about this promise is, after this Friday, once it's resolved, it will become either "fulfilled" or "rejected" and it would **remain in that resolved state forever**. Even if I look back at this promise after one year, the fact that I keep it \(or I don't\) does not change after this Friday.

It also worths highlighting that there are two parties/roles in this story:

* One party that creates a promise and finally decides when to resolve the promise, and if the promise should be fulfilled or rejected.
* The other party that receives the promise can register their action plans and wait to be notified on the outcome. The recipient/observer of a promise have no influence on whether the promise is fulfilled or rejected, nor can they decide when the promise would be resolved.

Let's write some codes using the promise API in ES6 to represent the example above. The sample codes below can be found in this [repository](https://github.com/thoughtworks-jumpstart/asynchronous-javascript-by-example)

We will first learn:

* How to create a new Promise
* How to notify the observers of the Promise when it's resolved \(either fulfilled or rejected\)

### Creating a new Promise

```javascript
const dinnerPromise = new Promise(function(fulfill, reject) {
  const waitTillThisFriday = 5000;
  setTimeout(() => {
    try {
      finishMyWorkOnFriday();
      const restaurant = bookRestaurant();
      const flower = buyFlower();
      fulfill({ restaurant, flower });
    } catch (badNews) {
      reject(badNews);
    }
  }, waitTillThisFriday);
});
```

#### How is the promise resolved/rejected?

Note that this Promise constructor takes in one callback function \(called **executor**\), which takes in two arguments: `fulfill` and `reject`.

The executor function basically does some **asynchronous task**, and invoke either `fulfill` or `reject` based on the success/failure of that asynchronous task. When one of those callbacks is called, the promise is changed from `pending` state to either `fulfilled` state, or `rejected` state. If there are any error thrown from the executor function, the promise is also rejected automatically.

#### What is this `fulfill` and `reject` parameter?

Those two parameters are callbacks supplied by the Promise library. If you are interested, you can have a look at the source codes of Promise constructor to see how it creates the `fulfill`/`reject` callback and supplies it to the `executor` function.

Since these two callbacks are supplied by the Promise library, you don't need to implement these two functions, your job is to implement the `executor` function and call `fulfill`/`reject` at a proper time!

#### Passing additional information when Promise is resolved/rejected

Note that we can pass some parameters to the `fulfill` callback. Those parameters would be available to those who subscribe to the success resolution event of the promise. We can also pass the error to the `reject` callback, which is then available to those who subscribe to the error notification of the promise.

#### The executor function should not take long to finish

Note that this Promise constructor calls the executor function right away \(before the Promise constructor function returns\). So typically you should not call any blocking API inside the executor function implementation. Otherwise, the Promise constructor call would be blocked.

### Observe the outcome of Promise

Now let's see how one can handle the outcome of a promise.

#### Register your action plan with `then` and `catch` function

Once you get hold of an instance of Promise, you can register your interests on the outcome via the `then` function available on the promise object.

```javascript
const onFulfilled = ({restaurant, flower}) => {...}
const onRejected = (badNews) => {...}

dinnerPromise.then(onFulfilled, onRejected);
```

The `then` function takes two arguments:

* The first argument is called `onFulfilled`, which is a callback function to handle the case when the promise is resolved. Note that the `onFulfilled` function receives the same list of arguments as received by the `fulfill` function in executor function.
* The second argument is called `onRejected`, which is a callback function to handle the case when the promise is rejected. The main purpose of this callback function is to handle the error scenario and bring the system back to normal state. Note that the `onRejected` function receives the same list of arguments as received by the `reject` function in executor function.

Although the `then` function can take in the `onRejected` event handler, sometimes people prefer to call `then` function with one argument only. In that case, the reject event needs to be caught by a call to `catch` function. e.g.

```javascript
dinnerPromise.then(onFulfilled).catch(onRejected);
```

#### Calls on Promise API can be chained

You may notice that in the code example above the calls on the promise instance are chained. We call `catch` function on the object returned from the `then` function. Why can we do it this way?

That's because all methods supported by this promise object returns another promise as return value. Specifically, those methods \(e.g. `then` and `catch`\) all behave in this way:

* If it finds a proper handler for the `fulfill` or `reject` event, it return a new promise which eventually resolves to the return value of the handler.
* If it does not find a proper handler for the `fulfill` or `reject` event \(e.g. when a promise is rejected and a call to `then` function does not supply `onRejected` callback\), the method would return the original promise.

#### Treating a Promise as a gift box

OK, this sounds pretty abstract. I have another mental model for you.

If you receive Promise object, you can think of a promise as a gift box. It's wrapped nicely, and you don't know what's inside. All you can see is there are two LED lights on the surface:

* A green LED light
* A red LED light

![promise as a gift box](../../.gitbook/assets/promise-gift-box.png)

When the promise is in `pending` state, neither LED light flashes.

When the promise is in `fulfilled` state, the green LED light flashes. There might be some value displayed on the panel, which reveals the secret hidden in the box.

When the promise is in `rejected` state, the red LED light flashes. There might be some vlalue displayed on the panel, which shows some error has happened.

When you call `.then` or `.catch` on the promise, you actually create another gift box like the picture shown below:

![creating another gift box](../../.gitbook/assets/promise-gift-box-chaining.png)

What the picture above shows are:

* When you call a `then` or `catch` method on a Promise, the function returns a new instance of Promise object \(i.e. a new gift box is created\).
* On the newly created gift box, there are also two LED lights, a green one and a red one.

The green LED light of the second gift box would flash under one of the three conditions:

1. If the green LED light of the first gift box flashes, and there is no `onFulfilled` handler in the second gift box, or
2. If the green LED light of the first gift box flashes, and the `onFulfilled` handler run to completion with no problem, or
3. If the red LED light of the first gift box flashes, and the `onRejected` handler run to completion with no problem.

You might wonder why the successful running of the `onRejected` handler lead to the green LED light of the second gift box to flash. Remember, the purpose of the `onRejected` handler is to handle the error and bring the system back to normal state. After the erratic behavior is detected and corrected by the `onRejected` handler, the error is fixed and the green LED light of the second box should flash.

The red LED light of the new box would flash under one of the three conditions:

1. If the green LED light of the first gift box flashes, and the `onFulfilled` handler throws error during execution, or
2. If the red LED light of the first gift box flashes, and there is no `onRejected` handler in the second gift box, or
3. If the red LED light of the first gift box flashes, and the `onRejected` handler throws error during execution.

Could the `onFulfilled` and `onRejected` handle throw error? Yes!

I can give you two examples here:

* The `onFulfilled` handler may return another promise object, which is eventually rejected. In that case, this is considered as error situation.
* The `onRejected` handler may encounter an error that it does not know how to handle, so it just throws it again and wishes another error handler can catch and handle it.

In either case, the red LED light of the second box would flash.

Note that the `onRejected` handler is optional when you call `then()`. If that is not specified, then the error reported by the first box cause the red LED light of the second box to flash right away.

Now you have a new gift box, you can pass it on to your friend, and similarly, they can create their own gift box by wrapping around your gift box, which result in a chain of gift boxes:

![gift box chaining](../../.gitbook/assets/promise-gift-box-chaining-2.png)

#### Promise Chaining Example

Let's look at another Promise chaining example using ES6 Promise API.

In the example below, we try to retrieve the skills of a user. There are two asynchronous operations involved:

1. find users by calling the `get()` API, which returns a Promise.
2. get a user's meta data by calling `getMetaDataFor()`, which also returns a Promise.

Since the second asynchronous operation depends on the output of the first asynchronous operation, we can chain them up using the Promise API.

```javascript
function getUserSkills(userId) {
  return users
    .get(userId)
    .then(user => {
      console.log(`Got user ${JSON.stringify(user)}`);
      return users.getMetaDataFor(user);
    })
    .then(userMetaData => {
      console.log(`Got metadata for user ${JSON.stringify(userMetaData)}`);
      return userMetaData.skills;
    })
    .catch(error => console.error(error));
}
```

If you find the codes above hard to understand, here is another version that written in synchronous/blocking style which you may be more familiar with:

```javascript
function getUserSkills(userId) {
  try {
    let user = users.get(userId); // assuming get() API is a blocking/synchronous API
    let userMetaData = users.getMetaData(user); // assuming getMetaData is a blocking/synchronous API
    let skills = userMetaData.skills;
    return skills;
  } catch (error) {
    console.error(error);
  }
}
```

In this synchronous version, you call multiple functions and each function use the result of calling the previous function.

The asynchronous version using the Promises achieves the same effect, except that they are using callbacks and not blocking.

#### Throwing errors from onFulfilled or onRejected handlers

If you ever call `throw error` yourself in the onFulfilled or onRejected handlers, or those handlers call some other function that throws errors, the call to `then` or `catch` would return a new promise that is rejected with the error received by the handler.

For example:

```javascript
function getUserSkills(userId) {
  return users
    .get(userId)
    .then(user => {
      throw new Error("Cannot find meta data for user", JSON.stringify(user));
    })
    .then(userMetaData => {
      console.log(`Got metadata for user ${JSON.stringify(userMetaData)}`);
      return userMetaData.skills;
    })
    .catch(error => console.error(error));
}
```

The first call to `then` function above would return a promise that is rejected with error. The second call to `then` does not provide any handler for rejection, so it's skipped and the event handler in `catch` would catch this error.

Note that this error is not thrown out of the `getUserSkills` function at all. It's handled by generating a console log. Then the error handler didn't return anything, so the caller of `getUserSkills` would get a Promise that's resolved to `undefined`.

#### onFulfilled or onRejected handlers are always executed asynchronously

When an the Promise implementation needs to invoke a onFulfilled or onRejected handler, it will always execute the handler in the next tick, i.e. it will do something like

```javascript
setTimeout(handler, 0);
```

That means those handlers supplied to `then` or `catch` call \(in the example above\) would NOT be called in the same call stack as then `then` or `catch` function.

Why? Because The ECMA 2015 spec declares that promises must not fire their resolution/rejection function on the same turn of the event loop that they are created on. This is very important because it eliminates the possibility of execution order varying and resulting in indeterminate outcomes.

In the example above, when `getUserSkills` function is executed \(and the `then`/`catch` functions are executed\), the `then`/`catch` function basically just register the event handlers in the event table, and return immediately. Later on, when an event handler needs to be executed \(because the corresponding Promise is resolved/rejected\), it will be put into the Event Queue and wait for its turn to run. When an event handler is executed by the JavaScript engine, it's already in a different call stack \(i.e. different from the call stack where `getUserSkills` function runs\).

Why do you need to understand this?

With this knowledge, you can understand why you cannot use a `try...catch` block inside the `getUserSkills` function to handle the error thrown from the `onFulfilled` or `onRejected` handlers.

e.g. the codes below does not work, because when the error is thrown from the event handler, the `getUserSkills` function has already finishes its execution and is removed from the call stack.

```javascript
function getUserSkills(userId) {
  try {
    return users
      .get(userId)
      .then(user => {
        throw new Error("Cannot find meta data for user", JSON.stringify(user));
      })
      .then(userMetaData => {
        console.log(`Got metadata for user ${JSON.stringify(userMetaData)}`);
        return userMetaData.skills;
      });
  } catch (error) {
    console.log("Cannot find meta data for user..", error);
  }
}
```

### Helper functions in Promise API

There are some handy utility functions on Promise API. You should check them out:

* [Promise.resolve\(\)](http://javascript.info/promise-api#promise-resolve)
* [Promise.all\(\)](http://javascript.info/promise-api#promise-all)

## A Use Case Study

Now, let's try some code with Promise API.

Many operations in JavaScript are asynchronous - for example: fetching data via HTTP, reading from a file, event handlers, etc.

In this example, we will use the [fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) provided by the browsers to fetch some data from a web service. The web service returns real time car park status information in Singapore.

The API we are going to call [display random pictures of dogs](https://dog.ceo/dog-api/)

Firstly visit [https://dog.ceo/api/breeds/image/random](https://dog.ceo/api/breeds/image/random) on your browser to see what the HTTP request should return.

Then look at this piece of code below, what do you think the value of `result` will be? Is it a JSON string containing the dog image URL?

Run this in the Chrome Developer console. Did the console output match your expectation?

```javascript
const result = fetch("https://dog.ceo/api/breeds/image/random");
console.log(result);
```

In the above example, you should find that `fetch` returns a `Promise`, i.e. `result` is a `Promise` object.

If you read the documentation of the [fetch\(\) API](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch), you can find that the returned value from the fetch API is "A Promise that resolves to a Response object.".

How can we retrieve data out of this [Response object](https://developer.mozilla.org/en-US/docs/Web/API/Response)? Again, based on the documentation of the Response, you can call the [response.json\(\)](https://developer.mozilla.org/en-US/docs/Web/API/Body/json) method to retrieve data sent in the response, assuming the data is in JSON format.

Let's try to call the `.json` method on the response object.

```javascript
fetch("https://dog.ceo/api/breeds/image/random").then(response =>
  console.log(response.json())
);
```

Hmm...if you look at the console log, the value returned from the `json()` method is still a Promise. If you read the [response.json\(\) documentastion](https://developer.mozilla.org/en-US/docs/Web/API/Body/json) again, it says it returns "A promise that resolves with the result of parsing the body text as JSON."

OK, now we know that we need to chain the promise call together, like the codes below:

```javascript
fetch("https://dog.ceo/api/breeds/image/random")
  .then(response => response.json())
  .then(json => console.log(JSON.stringify(json)));
```

## Common Mistakes

You will definitely makes some mistakes before you fully master the Promise API. Here are a few examples.

### Mistake: Unhandled Promise Rejections

One of the mistakes is you forget to handle errors that may be thrown from the promise.

If we remove the call to `catch` in the previous example:

```javascript
function getUserSkills(userId) {
  return users
    .get(userId)
    .then(user => {
      console.log(`Got user ${JSON.stringify(user)}`);
      return users.getMetaDataFor(user);
    })
    .then(userMetaData => {
      console.log(`Got metadata for user ${JSON.stringify(userMetaData)}`);
      return userMetaData.skills;
    });
}
```

If there are any errors thrown from the database lookup, or the `getMedataDataFor` function, the promise would become rejected. Since we don't handle the rejection here, and assuming nobody else handle this error, then it becomes an [unhandled promise rejection](http://thecodebarbarian.com/unhandled-promise-rejections-in-node.js.html)

### Mistake: Handle Rejected Promises with try..catch

You probably get use to handle errors in JavaScript using try..catch syntax, and you would like to handle errors from promises using that syntax too. Unfortunately, that won't work.

For example:

```javascript
function callMyPromise() {
  try {
    const userNamePromise = getUserSkills('gordon');
    userNamePromise.then(userName => console.log(userName)).catch(error) => throw error);
  } catch(error) {
    console.error("Failed in getting user name", error);
  }
}
```

That does not work because Promise implementation ensures the callbacks to `then` or `catch` is not executed in the current Call Stack \(i.e. the callbacks are always put back to the Event Queue first and executed in later ticks\).

This means the `callMyPromise` function would return before any callback handler passed to `then` or `catch` is called. The try...catch block here won't catch any error, and later when the error indeed happens, that error is thrown from the error handler function, and eventually becomes an unhandled promise rejection \(the `catch` call returns a promise that's rejected, and nobody handles that rejected promise.\)

### Mistake: Create a local instance of Promise and not return it

Look at the example below, what value would be returned from this function?

```javascript
function getUserSkills(userId) {
  users
    .get(userId)
    .then(user => {
      console.log(`Got user ${JSON.stringify(user)}`);
      return users.getMetaDataFor(user);
    })
    .then(userMetaData => {
      console.log(`Got metadata for user ${JSON.stringify(userMetaData)}`);
      return userMetaData.skills;
    });
}
```

Although there are some `return` statements in the codes above, don't be confused by them. They are the return statements from the callbacks, and not the return statements for `getUserSkills` function.

If we miss the `return` statement before `users.get..`, the caller of this function would get `undefined` result.

Another example of forgetting returning the promise is in the example below:

```javascript
test("the fetch fails with an error", () => {
  expect.assertions(1);
  expect(fetchData()).rejects.toMatch("error"); // we are missing return statement here if `fetchData` returns Promise
});
```

Assuming the `fetchData` function returns a Promise. This is a [test case for some asynchronous function written in Jest](https://facebook.github.io/jest/docs/en/asynchronous.html). We need to use the `return` statement so that Jest knows this test calls an asynchronous function and it needs to wait for the promise to resolve before it end the test function.

Without the `return` statement, the test case would finish without running the `fetchData` at all.

### Mistake: Forget to wait for a promise function to resolve

Sometimes, you may call a function that returns promise, but you forget about this fact and use it like a synchronous function \(i.e. assuming once the function returns, all the task is done\). This can leads to bugs that are hard to find.

For example:

```javascript
db.connect();
db.execute_statement(...);
```

Assuming the `connect` API returns a promise, and when that promise is resolved, you can start to execute statement with the database.

The code above does not work because when the `execute_statement` is called, the database connection may not be established yet.

So the correct way of using the API should be

```javascript
db.connect().then(() =>
  const execution = db.execute_statement(...);
  execution.then(...).catch(...)
);
```

### Mistake: Missing the return statement in onFulFilled or onRejected handlers

Remember the returned result from `onFulfilled` or `onRejected` handler is used as argument for the next handler in the promise chain, so if some handler forget to return anything, then the next handler would get `undefined` in their arguments.

For example:

```javascript
function getUserSkills(userId) {
  return users
    .get(userId)
    .then(user => {
      console.log(`Got user ${JSON.stringify(user)}`);
      users.getMetaDataFor(user); // missing return statement here
    })
    .then(userMetaData => {
      console.log(`Got metadata for user ${JSON.stringify(userMetaData)}`);
      userMetaData.skills; // missing return statement here
    })
    .catch(error => console.error(error));
}
```

Note that the result from `getMetaDataFor` function is not returned, that means the next handler which is expecting `userMataData` as the argument would get `undefined` parameter value.

## Other Promise Implementations

After this `Promise` idea was proposed, there were quite a few open source implementations before it's supported in ES6 by default. One of them is [bluebird](http://bluebirdjs.com/docs/getting-started.html) and another one is called [Q](http://documentup.com/kriskowal/q/). Most of them support similar API as the one offered in ES6, and we can even mix them together.

## How to convert API that only support callbacks to return a Promise instead

We see promises are useful tools to handle asynchronous tasks, but how can we deal with existing functions that are written in a way to take callback as arguments?

Node.js comes with a utility function called `promisify`, which takes a regular callback-taking function as an argument and returns its promisified version. e.g.

```javascript
const { promisify } = require("util");
const { readFile } = require("fs");
const readFilePromise = promisify(readFile);

// Original callback-style readFile
readFile("path/to/file", "utf8", (err, data) => {
  if (err) {
    console.error(err);
    return;
  }

  console.log(data);
});

// Promisified version
readFilePromise("path/to/file", "utf8")
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.error(error);
  });
```

* util.promisify docs: [https://nodejs.org/api/util.html\#util\_util\_promisify\_original](https://nodejs.org/api/util.html#util_util_promisify_original)
* More information on how util.promisify works: [http://2ality.com/2017/05/util-promisify.html](http://2ality.com/2017/05/util-promisify.html)

## Resources

* [Promise Basics](http://javascript.info/promise-basics)
* [Async JavaScript Cheatsheet](https://github.com/frontarm/async-javascript-cheatsheet)
* [Promise Chaining](http://javascript.info/promise-chaining)
* [Promise API](http://javascript.info/promise-api)
* [Three ways of understanding promises](http://2ality.com/2016/10/understanding-promises.html)
* [Promises in ES6 \(youtube video\)](https://www.youtube.com/watch?v=2d7s3spWAzo)
* [Escape from the Callback Mountain](https://github.com/justsml/escape-from-callback-mountain)
* [JavaScript Promises with Node.js](https://itnext.io/javascript-promises-with-node-js-e8ca827e0ea3)
* [Promise Fun](https://github.com/sindresorhus/promise-fun)
* [Promise Tips](https://dev.to/kepta/promising-promise-tips--c8f)
* [JavaScript Promises for Dummies](https://scotch.io/tutorials/javascript-promises-for-dummies)
* [Write your own promise library from scratch](http://thecodebarbarian.com/write-your-own-node-js-promise-library-from-scratch.html)
* [Master the JavaScript Interview: What's a Promise](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261)

## Lab

Here is a useful workshop that illustrates the basics of promises. Follow the instructions step by step to get some hands-on exercises on Promise.

* [Promise it won't hurt](https://github.com/stevekane/promise-it-wont-hurt)

Note:

* In the second step of the workshop, it requires you to install a library `es6-promise` and require it in your code. That's not necessary anymore because the latest Node version already have built in support for Promise.
* In the step on "Fetch JSON", the URL mentioned in the instruction should have been "[http://localhost:1337](http://localhost:1337)"

