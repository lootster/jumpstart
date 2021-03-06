# async and await

## What is it?

Although Promise API helps to avoid the callback hell, it's still requires you to think in asynchronous style. How nice if we could write those asynchronous functions in synchronous style!

If you feel that way, you are not alone: `async/await` allows you to do exactly that.

> Async function make it possible to treat functions returning Promise objects as if they were synchronous.

For example, our previous example on Promise looks like this

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

Using `async/await` syntax, the same function can be written in the format below: \(Do you feel the version below is easier to understand compared with the Promise version?\)

```javascript
async function getUserSkills(userId) {
  const user = await users.get(userId);
  console.log(`Got user ${JSON.stringify(user)}`);

  const userMetaData = await users.getMetaDataFor(user);
  console.log(`Got metadata for user ${JSON.stringify(userMetaData)}`);

  return userMetaData.skills;
}
```

Note the two new keywords we use above:

* `async`: it marks the function as an async function
* `await`: it means we want to wait for the Promise \(returned by the function after await\) is resolved before the next line is executed

## What is returned from an `async` function?

In the example above, you may think the async function return a string \(which is the user name\), but remember the async function above is the same as the promise version, and the promise version returns an instance of Promise to the caller.

An `async` function returns an instance of a Promise that resolves to the return value as you put in the `return` statement of the function.

That means, if you call an `async` function, then you need to either call `.then()` on the returned result (because it's a promise), or you need to use `await` to extract the value from the promise object.

For example,

```javascript
getUserSkills("gordon").then((skills) => ...)
```

```javascript
const skills = await getUserSkills("gordon");
```

## Error Handling within Async Functions

A nice feature of async function is you can handle the error with try-catch block again. Remember, you cannot call `throw` in the Promise handlers and catch those errors with try-catch. But with the async functions, you can do that again.

```javascript
async function getUserSkills(userId) {
  try {
    const user = await users.get(userId);
    console.log(`Got user ${JSON.stringify(user)}`);

    const userMetaData = await users.getMetaDataFor(user);
    console.log(`Got metadata for user ${JSON.stringify(userMetaData)}`);

    return userMetaData.skills;
  } catch (error) {
    console.error(error);
    // this is just to demonstrate we can use try-catch block here to catch the errors
    // actually it's not a good idea to return null
    return null;
  }
}
```

In this case, if the promise returned by `users.get(userId)` or `users.getMetaDataFor(user)` is rejected, `await` will translate that into normal Error that you can `catch` hand handle.

## Understanding `await` using the Gift Box model

The gift box model we introduced in the last section can also help us to understand the effect of `await` on a Promise object.

`await` basically unwrap the gift box and shows you what's inside.

* If the promise is fulfilled, you will see the hidden secret value.
* If the promise is rejected, then you open a box with a bomb and the bomb blows when the box opens. To protect yourself against such scenario, you need to use try...catch block to catch the errors.

## What if there are errors thrown directly from an Async Function?

If you try to throw error from an Async function, can it be handled with try-catch when you call that async function? The answer is 'No'.

For example, the async function below contains a throw statement:

```javascript
async function testAsyncError() {
  throw new Error("Test Errors");
}
```

When we call the `testAsyncError` function, we cannot use try-catch \(i.e. the code below does not work\):

```javascript
function thisDoesntWork() {
  try {
    testAsyncError();
  } catch (error) {
    console.error(error);
  }
}
```

Why?

Because **An async functions always return a Promise object**. If there are errors thrown from the function, the async function returns a rejected promise instead of throwing the error directly.

That means you have to handle the error using promise-style:

```javascript
  testAsyncError().catch(error => {...});
}
```

Or use async/await syntax all the way:

```javascript
async function thisWorks() {
  try {
    await testAsyncError();
  } catch (error) {
    console.error(error);
  }
}
```

Note: don't forget about the `await` keyword in the example above. If you remove it \(like the one below\), does it still work?

```javascript
async function doesThisWork() {
  try {
    // missing await here
    testAsyncError();
  } catch (error) {
    console.error(error);
  }
}
```

## `await` can ony be used within a function

Since `await` can only appear inside a function that is marked as `async`, you can't have `await` in top-level code \(i.e. code that is not inside any other function and is invoked directly when the program runs\).

```javascript
// syntax error in top-level code
const response = await fetch('/path/to/some/json');
```

It needs to be wrapped in another function marked as `async`, which is then invoked:

```javascript
async function getData() {
  const response = await fetch('/path/to/some/json');
}

getData();
```

## Can I use it now?

`async/await` is a part of ES7, and is [supported in the latest versions of main-stream browsers](https://caniuse.com/#feat=async-functions). It's also supported by Node.JS platform since 7.6.0 release.

## Resources

* [Brief guide to async/await](https://javascript.info/async-await)
* [Intro to `async` and `await`](https://developers.google.com/web/fundamentals/primers/async-functions)
* [Async Await Tutorial](https://www.toptal.com/javascript/asynchronous-javascript-async-await-tutorial)
* [Yet Another Async Await Tutorial](https://davidwalsh.name/async-await)
* [6 reasons why javascript async/await blows promises away](https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9)
* [The 80/20 guide to async/await in node.js](http://thecodebarbarian.com/80-20-guide-to-async-await-in-node.js)
* [Write cleaner route handlers with async/await](https://itnext.io/using-async-await-to-write-cleaner-route-handlers-7fc1d91b220b)
* [How to escape async/await hell](https://medium.freecodecamp.org/avoiding-the-async-await-hell-c77a0fb71c4c)

