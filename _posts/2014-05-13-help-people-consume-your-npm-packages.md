---
layout: post
category: web
tags: npm, node.js
title: Help People Consume Your npm Packages
---

Today I want to share with you some pain and thoughts on current state of [npm](https://www.npmjs.org/) packages and `node_modules` folder in your application root.

Most of the times npm packages is declared as dependencies of your app in `package.json` and installed every time you need them. Generally there should not be much more than single `npm install` for each working copy of your project.

## The Pain
But things get complicated when you decide to use continuous integration/deployment, where your project is constantly built from scratch. Our TeamCity instance builds about 150 times a day on average and npm packages is installed every single time. Usually it takes about 30 seconds to <strike>Mars</strike> install all packages from local npm mirror. That is about **75 minutes** every day is spent on repeated installs of same npm packages. Ouch. I think that's a lot of time and potential bottleneck.

Moreover, npm itself is not very stable and I often end up resolving some weird unclear and hardly reproducible portability issues, cleaning caches and reinstalling different versions. Just recently there was an network-related accident with our registry mirror, at the same time several broken packages were published on npmjs.org, European mirror had some outdated packages and npm@1.4.7 somehow become extremely buggy at our environment. An entire team was unable to work for several hours. And this really makes me upset.

## Alternative View
I came across [an old article](http://www.futurealoof.com/posts/nodemodules-in-git.html) where Mikeal Rogers talks about the reasons for committing your `node_modules` into git. For the first time this may sound crazy to you, but the justifications he gives is very strong. And if you're still not convinced, take a look at [this question on StackOverflow](http://stackoverflow.com/questions/11459475/should-i-check-in-node-modules-to-git-when-creating-a-node-js-app-on-heroku
). Just in case you haven't noticed, all three top-voted answers recommend the same. Even [npm documentation](https://www.npmjs.org/doc/faq.html#Should-I-check-my-node_modules-folder-into-git) agree with this approach!

Actually, original article was about fixing dependencies of your dependencies and now there is [npm shrinkwrap](https://www.npmjs.org/doc/cli/npm-shrinkwrap.html) to the rescue. But it still [does not give](http://stackoverflow.com/questions/11459733/check-in-node-modules-vs-shrinkwrap) you full confidence in deployment. And I decided to try and commit my `node_modules` folder into git. You know what?

## Authors Doesn't Care
Some npm packages are so huge that committing them becomes scary:

  * [bower](http://bower.io/) installation takes up to 31 megabytes on your drive! OMG, is this real? Please tell me that I need all that stuff. node.js itself is only 11 megs and npm is 10 MB! I'm gonna switch!
  * [grunt](http://gruntjs.com/) is 6 MB. I love you guys, but I really don't need CoffeeScript there.
  * [mocha](http://visionmedia.github.io/mocha/) is 1.5 megs, but my TeamCity doesn't care that it has jade and growl.
  * [express](http://expressjs.com/) is 872 KB. Well, that's great, isn't it? But I believe you can do it even smaller.

No offense! I hope that you can see the trend now. Dear contributors, please don't forget how much space and bandwidth is consumed by your library!

Think of it as a real life product such as notebook with pre-installed Windows or almost any modern smartphone. They often have dozens of useless builtin branded crap like audio players, image editors or Chinese keyboards which you'll try to uninstall as quickly as possible and continue to use good old whatever-amp. But usually you are doomed to live with this [bloatware](http://en.wikipedia.org/wiki/Software_bloat#Bloatware) forever.

## Anatomy of npm Package
Let's now talk about useful payload of your npm package.
I asked this question on [StackOverflow](http://stackoverflow.com/questions/23090677/what-should-one-put-into-npm-package) but it has not received as much attention as I would have liked.

Speaking in terms of object-oriented design, your npm package should be [highly cohesive](http://en.wikipedia.org/wiki/Cohesion_(computer_science)) and [loosely coupled](http://en.wikipedia.org/wiki/Loose_coupling). Your package should do only one thing and do it well.

So, what should you put into npm package?

**README.*** (if any) and **package.json** can't be ignored so you just can't skip them. Next thing is **library core functionality**, the stuff people install your lib for. No bells and whistles. If your package is about trimming strings, please don't make it occasionally brew beer. No doubt, it's great and useful, but you should put it in another `brewmaster` package. And one more thing&hellip; oh, there's not. That's all we need!

From the other side, here are the things that have no place in your package:

  * **Dev-only files** like `Gruntfile.js`, `.jshintrc`, `.travis.yml` and so on. Nobody needs your code style config in their app. Also make sure that external dev-only packages are listed in `devDependencies` instead of regular `dependencies`.
  * **Tests**. You should not include tests in your distribution package unless your library is heavily depend on platform and it's behavior should be verified.
  * **Different versions of the same functionality**. You better show me how to build one from source by myself.
  * **Hosted docs, logos examples and other assets.** [Beautiful logos](https://github.com/andris9/Nodemailer/tree/master/assets), nodemailer! Sorry, I can't take it with me, it adds another 200 KB to my repo.
  * **Other stuff** that can be optional: plugins, contribs, rulesets etc.

Use your `.npmignore` wisely. Start with:

```
*

!lib/**
!bin/**
!index.js
```

You can also use `files` and `directories` fields in `package.json`. Some helpful links to read:

  * [npm Developers Guide](https://www.npmjs.org/doc/misc/npm-developers.html)
  * [npm package.json description](https://www.npmjs.org/doc/files/package.json.html)

## Example
As a recent example please have a look at [teamcity-service-messages](https://github.com/pifantastic/teamcity-service-messages) (don't forget to star it!). This package has no external dependencies and installed instantly:

```
teamcity-service-messages
├── LICENSE.txt
├── README.md
├── index.js
├── lib
│   └── message.js
└── package.json
```

There is nothing extra.

Underscore.js is doing good job too:

```
underscore
├── LICENSE
├── README.md
├── package.json
├── underscore-min.js
└── underscore.js
```

Wonderful! 5 files, 76 KB only! I'm ready to commit it right away!

## Conclusion
**Libs developers**. Please, include only vital components of your library to npm packages. Review your payload and deps now.

**App devs**. Please, remove `node_modules` from your `.gitignore`. Make your development and deployment a little bit simpler and faster.

What do you think about this? Have you experienced similar problems? Have you ever thought about committing your `node_modules` into git? How can we help each other?
