---
layout: post
category: web
title: Loops and Promises
---

If you ever done [Node.js](http://nodejs.org/) development then you probably know that server side JavaScript is filled with async operations (like HTTP requests, file access and other I/O) and lots of callbacks. I don't have much experience with Node and when I was asked to implement a command line utility with node, I really stuck with asynchronous loops. And this is what I want to talk to you today.

The task was pretty simple: we need to start a remote process, poll the API until completion and process the results. Now this may look silly for you, but back in time it took me couple of hours to figure how to code this properly.

Let's first look at polling loop with callback implementation:

```js
function getStatus(onSuccess) {
  // some async code that eventually calls `onSuccess`
}

function waitForCompletion(onSuccess) {
  setTimeout(function() {
    getStatus(function(status) {
      if (status === 'finished') {
        onSuccess(status);
      } else {
        waitForCompletion(onSuccess);
      }
    })
  }, 5000);
}
```

It works. Not so bad, but there is a code smell which is commonly referred as [callback hell](http://callbackhell.com/). You can imagine how fast our code can grow in width if we continue to use anonymous callbacks. You could also spot that we have no error handling at all, which is not good by itself and in turn could also add us more lines here.

There are some techniques that can help you deal with callback hell and one of them is called promises. Promise is an abstraction on async operation that could be resolved or rejected with some value only once and combined with chaining could make asynchronous code look like synchronous. [Q](https://github.com/kriskowal/q) is the most popular promise library I know.

So now we implement the same logic using promises:

```js
function getStatus() {
  // some async code that returns a promise
}

function waitForCompletion() {
  return Q.delay(5000).then(getStatus).then(function(status) {
    return status === 'finished' ? status : waitForCompletion();
  });
}
```

This looks shorter and much cleaner, isn't it? It actually does what it says: waits 5 seconds, asynchronously gets the status and then verifies it. If it is ok, then the promise from `waitForCompletion` will be fulfilled with `status`, otherwise another polling promise will be returned and evaluated.

And finally, here is a little code snippet that shows how to implement async operation with several retries on failure:

```js
function httpGetWithRetries(url, maxRetries, retry) {
  // first try doesn't count as retry, initialize with zero
  retry || (retry = 0);

  // httpGet returns a promise
  return httpGet(url).fail(function (err) {
    // fail after maxRetries
    if (retry >= maxRetries)
      throw err;

    // wait some time and try again
    return Q.delay(5000).then(function () {
      return httpGet(url, maxRetries, ++retry);
    });
  });
}
```

Hope this article will help you! Maybe you know other Promise patterns worth sharing? By the way, you might also want to have a look at [Promise Anti-patterns](http://taoofcode.net/promise-anti-patterns/) too.

Have a nice day!
