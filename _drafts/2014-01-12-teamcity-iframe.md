---
published: false
layout: post
category: ci
---

About the problem
There are many tools that produces HTML reports
But publishing additional artifacts on every build will consume your space

## `iframe` trick
There is a neat workaround.

1. Make sure you have configured custom report tab ([refer to TeamCity docs](http://confluence.jetbrains.com/display/TCD8/Including+Third-Party+Reports+in+the+Build+Results)). Usually you can specify `index.html` and reuse it in all your projects.
2. Create new build configuration and add `%TargetFile%` as artifact path. There is no VCS roots to set up.
3. In a build step creation choose command line runner. Select "Run custom script" and

```sh
echo '<script>location="%TargetUrl%"</script>' > %TargetFile%
```

You can also use version without JavaScript:
```html
echo '<meta http-equiv="refresh" content="0; url=%TargetUrl%" />' > %TargetFile%
```
But I prefer the former because it clean and short enough for my needs.

See http://confluence.jetbrains.com/display/TCD8/Typed+Parameters to configure your parameters.

Don't forget to include protocol in your URLs (e.g. http) because iframe will treat URLs without protocol relative to current domain.

And here is the result:


## Metarunners
If you are using TeamCity 8 then you can easily reuse configuration you just created  extracting a metarunner from it. This task is relatively easy and I just send you [to documentation](http://confluence.jetbrains.com/display/TCD8/Working+with+Meta-Runner).

## Conclusion
Custom report tabs is great functionality but the kind of limited for now. In TeamCity 8.1 report tabs may be configured for each project separately which is nice improvement.
You can report custom statistics like SonarSource dashboard