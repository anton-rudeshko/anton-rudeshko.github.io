---
layout: post
title: TeamCity tips
category: ci
---
I was working with TeamCity quite a lot for now and want to share some tips I've learned.

## Branch name display
TeamCity is automatically assigning display names for your branches, but you can alter this by manually
setting the `teamcity.build.branch` parameter in your build configuration.

For example if you dealing with several projects in single build configuration
you have to somehow distinguish them from each other.
One way is to define this parameter using values of other parameters:

```
teamcity.build.branch: %project.name%#%project.branch%
```

And here is the result:

![Custom branch names](/assets/custom_branch_names.png)

I believe that this can be even done in the runtime using
[Service Messages](http://confluence.jetbrains.com/display/TCD8/Build+Script+Interaction+with+TeamCity).

## GitHub build status reporting
There is a [great plugin](https://github.com/jonnyzzz/TeamCity.GitHub)
that connects TeamCity builds with git commits on GitHub.
One of the features is that TeamCity will report build status to pull request page (and here is a
[comprehensive blog post about it](http://blog.jetbrains.com/teamcity/2013/02/automatically-building-pull-requests-from-github-with-teamcity/)).

But there is one caveat: your build description length should not exceed tweet size (140 characters).
This is not actually documented anywhere and TeamCity reporting will silently fail with following error:

```
"description is too long (maximum is 140 characters)"
```

The sad thing that this is can only be observed in TeamCity debug logs
and on pull request discussion page you will see permanent "Build pending".

Let's look on how the plugin is constructing build status description:
fixed text "TeamCity build" followed by full build configuration name, "finished" and its finish status:

![Long pull request description](/assets/long_pr_desc.png)

That's it, be nice and don't use a lot of long-named nested projects with large messages in their finish status.

## One more thing about reporting
When you successfully integrate GitHub reporting you'll probably want it everywhere (because its awesome).
But it will not work for builds in chains that uses snapshot dependencies.
For every build configuration in the chain you should configure same VCS root as in your first snapshot dependency
([vote for issue](https://github.com/jonnyzzz/TeamCity.GitHub/issues/30)).

## Pull requests and default branch
Usually your main development branch run much more tests that is necessary
for feature branches and pull requests, so you would probably separate them into different configurations.

But now you have to exclude your main branch from building in PR-dedicated configuration.
There is a great feature was introduced recently in VCS Trigger – Branch filters – where you can safely exclude default branch:

```
+:*
-:<default>
```

But then you will notice that TeamCity stacks all already merged changes into each pull request and
report much more changes than it actually exists in feature branch.
This can lead to notifying wrong people about failure.

As a workaround you should first explicitly check for `%teamcity.build.branch.is_default% == "true"` in your build steps.

There is a issue for this one too ([TW-33429](http://youtrack.jetbrains.com/issue/TW-33429)).

Now go and start using continuous integration for your team!

Thanks for reading and have a nice day!
