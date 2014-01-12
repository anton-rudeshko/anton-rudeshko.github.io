---
published: false
layout: post
category: ci
---

About the problem
There are many tools that produces HTML reports
But publishing additional artifacts on every build will consume your space

## Solution

%TargetFile% as artifact
There is no VCS roots to set up.
Choose command line runner. Selec "Use custom script"
Specify working directory if needed.
You need a admin role to create

http://confluence.jetbrains.com/display/TCD8/Including+Third-Party+Reports+in+the+Build+Results

```sh
echo '<script>location="%TargetUrl%"</script>' > %TargetFile%
```

You can also use version without js:
```html
echo '<meta http-equiv="refresh" content="0; url=%TargetUrl%" />' > %TargetFile%
```
But I prefer the former because it clean and short enough.

See http://confluence.jetbrains.com/display/TCD8/Typed+Parameters to configure your parameters.

Don't forget to include protocol in your URLs (e.g. http) because iframe will treat URLs without protocol relative to current domain.

## Metarunners
If you are using TeamCity 8 then this could be even easier. And select "Extract metarunner". It will promt you with several parameters.

http://confluence.jetbrains.com/display/TCD8/Working+with+Meta-Runner

## Conclusion
You can report custom statistics like SonarSource dashboard