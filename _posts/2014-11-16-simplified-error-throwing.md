---
layout: post
category: web
tags: javascript, node.js
title: Simplified Error Throwing in JavaScript
---

Throwing errors is hard. Bad errors give you useless information and mislead you. Good errors help you debug your code and suggest possible fixes. I highly recommend you to read a post by Nicholas Zakas on this subject: [The Art of Throwing JavaScript Errors Part 1](http://www.nczonline.net/blog/2009/03/03/the-art-of-throwing-javascript-errors/) (why throw and when to throw) and [Part 2](http://www.nczonline.net/blog/2009/03/10/the-art-of-throwing-javascript-errors-part-2/) (what to throw). Today I would like to discuss practical part of throwing errors.

JavaScript is a very flexible language. As my experience grows my code tend to be more elegant and concise. I try to express more in less. I often use functional programming concepts like [currying](http://en.wikipedia.org/wiki/Currying) (see also [Why Ramda?](http://fr.umio.us/why-ramda/)) and shortcuts of conditionals with boolean operators (`&&` and `||`, to get more sense about using JavaScript logical operators as conditionals check out [this article](http://addyosmani.com/blog/exploring-javascripts-logical-or-operator/) by Addy Osmani).

I used to read and write code like this:

```js
var port = process.env['PORT'] || 3000;
DEBUG && console.log('Port is', port);
```

But how do we usually throw errors? For example, we might want to check that required environment variable is set:

```js
var port = process.env['PORT'];

if (!port) {
    throw new Error('PORT is not specified');
}
```

As you may know, JavaScript have expressions and statements. Axel Rauschmayer in [Expressions vs Statements](http://www.2ality.com/2012/09/expressions-vs-statements.html) tells us:

> Wherever JavaScript expects a statement, you can also write an expression. [&hellip;] The reverse does not hold: you cannot write a statement where JavaScript expects an expression.

One of the disadvantages of `throw` is that it is not an expression and you can't use it in expression context:

```js
var port = process.env['PORT'] || throw new Error('PORT is not specified');
// SyntaxError: Unexpected token throw
```

That bothered me so much that I decided to just wrap `throw` in a function (so that it becomes an expression) and drop it into [npm module](https://www.npmjs.org/package/throw) ([on GitHub](https://github.com/anton-rudeshko/node-throw)) to reuse in my projects:

```js
var thr = require('throw');

var port = process.env['PORT'] || thr('PORT is not specified');
```

It is very simple module yet extremely useful. It also utilizes printf-like arguments and you can get rid of string concatenation when constructing error messages:

```js
function getEnvVar(variableName) {
    return process.env[variableName] || thr('Environment variable "%s" is required', variableName);
}

var parsed = parse(input) || thr('Could not parse', input);

function firstMatch(regex, input) {
    var match = regex.exec(input) || thr('%s does not match %s', input, regex);
    return match[1];
}
```

Maybe it help you too.

Currently custom error types are not supported in node-throw, but they may eventually (file an issue with use case or just send me a pull request).

Some additional links:

  * [Exceptions: Why throw early? Why catch late?](https://programmers.stackexchange.com/questions/231057/exceptions-why-throw-early-why-catch-late)

Have a nice day!