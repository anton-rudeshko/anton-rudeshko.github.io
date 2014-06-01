---
layout: post
category: ci
tags: node.js
title: Accessing TeamCity Build Parameters from Node.js
---

To avoid copy-pasting build configurations in [TeamCity](http://www.jetbrains.com/teamcity/) and keep them [DRY](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself) you should use [configuration templates](http://confluence.jetbrains.com/display/TCD8/Build+Configuration+Template) and build parameters. It's easy when you're working with Java or .NET build scripting tools such as [Ant](http://ant.apache.org/), [Maven](http://maven.apache.org/) or [MSBuild](http://msdn.microsoft.com/en-us/library/wea2sca5(v=vs.90).aspx) because TeamCity have a first class support for those and you can pass build parameters around with no effort. But same task may become more difficult in other development areas.

## CI in Frontend

In frontend development our primary continuous integration tools for now is Bash and Node.js scripts. In TeamCity we use [command line runner](http://confluence.jetbrains.com/display/TCD8/Command+Line) to execute our scripts. There we can access all build parameters (including configuration) through percent references: `%project.name%`. It's that simple, but mostly uncomfortable and unreliable because of textarea editor without syntax highlighting and linting:

![Textarea editor](/assets/2014-06-01-accessing-build-parameters-from-node/textarea-editor.png)

We've also tried to build simple CLI applications with arguments and options (see [coa](https://github.com/veged/coa/) module, for example). Then our shell runners began to look like this:

```sh
node my-script.js --name=%project.name% --my-service-url=%myService.url%
```

Much better, but it still daunting to write this again and again and maintain unnecessary CLI interface in every single script.

## Making It Work with Node.js

And recently we've implemented a very simple npm module that provides seamless access to TeamCity build properties.

For every build TeamCity creates a temporary `.properties` file on the agent file system with all build properties stored in it and puts absolute path to this file in `TEAMCITY_BUILD_PROPERTIES_FILE` environment variable. Then we can parse that with [node-properties](https://github.com/gagle/node-properties) and export for further use. And this is basically how it works!

You can found it on [GitHub](https://github.com/anton-rudeshko/teamcity-properties) or on [npm registry](https://www.npmjs.org/package/teamcity-properties).

Install it:

```sh
npm install --save teamcity-properties
```

I also encourage you to commit your `node_modules` into your VCS [to simplify deployment](/web/2014/05/13/help-people-consume-your-npm-packages.html) of your app and tools.

Use it:

```js
var tcProps = require('teamcity-properties');

// not fail-safe
var agentName = tcProps['agent.name'];

// throws if no such property
var projectName = tcProps.get('myCompany.project.name');
```

Pretty straight forward. The only caveat is that it's very important to understand the difference between three types of build parameters and when they should be used. To recall, I'll just quote the documentation:

>Environment variables (defined using `env.` prefix) are passed into the spawned build process as environment.

>System properties (defined using `system.` prefix) **are passed into the build scripts** of the supported runners (e.g. Ant, MSBuild) as build-tool specific variables.

>Configuration parameters (no prefix) **are not passed into the build** and are only meant to share settings within a build configuration. They are the primary means for customizing a build configuration which is based on a template or uses a meta-runner.

So here we are simply making Node.js scripts understand TeamCity system properties, that's all. All your custom parameters that you want to use inside your build scripts should be defined in **System Properties** section:

![Build parameters](/assets/2014-06-01-accessing-build-parameters-from-node/build-parameters.png)

And accessed without `system.*` prefix:

```js
var myServiceUrl = tcProps.get('myService.url'); // http://example.org
```

You can read more about configuration parameters [in TeamCity docs](http://confluence.jetbrains.com/display/TCD8/Configuring+Build+Parameters).

## Conclusion

This solution allowed us to move all of our Node.js deployment scripts and tools from TeamCity into separate repository and get rid of complex command line interfaces and other superfluous interlayers.

In the next post I'll tell you about accessing build parameters from Bash scripts.

Please share your thoughts and suggestions in the comments below.

Thanks for reading! Go now and automate all the things! Have a nice day!
