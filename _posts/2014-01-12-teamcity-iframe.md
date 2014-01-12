---
layout: post
category: ci
title: External reports in TeamCity
---

There are quite a few developer tools around which produces HTML reports (QUnit, Jasmine) and
often you want to run them on schedule in TeamCity and see the results there too.
[TeamCity already provides][custom-reports] useful functionality to handle this: you may publish build-generated HTML pages and
tell TeamCity to show them in additional tabs on the build overview page.

But sometimes reports may be already published somewhere on the network and publishing additional artifacts on
every build may consume much of your space (for example, if they contain a lot of pictures).
So I want to share a little trick with you here which will help you reuse existing reports located on external resources.

## Redirect trick
There is a neat and simple workaround.
You can actually give TeamCity an HTML page with a redirect in it.
All we need is just pass a couple parameters inside it.
Let me show you.

First, make sure you have configured custom report tab at Administration &rarr; Integration &rarr; Report Tabs ([see docs][custom-reports]).

![Setting up custom report tab][report]

Usually you can specify just `index.html` and reuse it in all your projects.

Then we create a new build configuration.
On the "General Settings" tab add `%TargetFile%` as artifact path and skip following VCS root set up.

![Setting up general build options][general]

Choose command line runner and "custom script".
Use following bash snippet, it will redirect TeamCity report iframe to any page of your choice:

```sh
echo '<script>location="%TargetUrl%"</script>' > %TargetFile%
```

![Configuring build step](/assets/teamcity-iframe/step.png)

You can also use version without JavaScript or both of them for sure:

```sh
echo '<meta http-equiv="refresh" content="0; url=%TargetUrl%" />' > %TargetFile%
```

But I prefer the first one because it shorter and cleaner and works well.

On the "Build Parameters" tab you can [label your parameters and add a description][params].

![Target file parameter configuration][file-param]

![Target url parameter configuration][url-param]

![Target url parameter specification][url-spec]

Don't forget to include protocol in your URLs (e.g. **http://**www.google.com instead of www.google.com)
because iframe will treat those as relative to current page.
Note that you can also use two slashes only (`//`, [protocol-relative URL][protocol]).

That's all! Test run:

![Test run][run]

And here is the result on the build overview page:

![Browsing WebPlatform.org in TeamCity][result]

Hope it will help you!

## Meta Runners
If you are using TeamCity 8+ then you can easily reuse created configuration through extracting a meta runner from it.
This task is relatively easy and I prefer to just send you [to the docs][meta-runner].

## Conclusion
Custom report tabs is great feature but they kind of limited for now.
In TeamCity 8.1 report tabs may be configured for each project separately which is a nice improvement because
you no longer need to be a server admin to create them.
On project level you can display custom overall statistics like [Sonar][sonar] inspections dashboard or website speed charts from any source.
I hope that someday JetBrains will allow displaying reports by URLs in TeamCity so that there will be no need to use this workaround anymore.

Thanks for reading, have a nice day!

[custom-reports]: http://confluence.jetbrains.com/display/TCD8/Including+Third-Party+Reports+in+the+Build+Results
[params]: http://confluence.jetbrains.com/display/TCD8/Typed+Parameters
[meta-runner]: http://confluence.jetbrains.com/display/TCD8/Working+with+Meta-Runner
[protocol]: http://www.paulirish.com/2010/the-protocol-relative-url/
[sonar]: http://www.sonarsource.com/

[report]: /assets/teamcity-iframe/report.png
[general]: /assets/teamcity-iframe/general.png
[file-param]: /assets/teamcity-iframe/file-param.png
[url-param]: /assets/teamcity-iframe/url-param.png
[url-spec]: /assets/teamcity-iframe/url-param-spec.png
[run]: /assets/teamcity-iframe/run.png
[result]: /assets/teamcity-iframe/result.png
