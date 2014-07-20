---
layout: post
category: ci
tags: bash, teamcity
title: Accessing TeamCity Build Parameters from Bash
---

In the [last post](/ci/2014/06/01/accessing-build-parameters-from-node.html) we talked about how one can access and use TeamCity build parameters from Node.js. Today we will look at the same problem from another side – Bash scripts.

## A little helper

This code is not invented by me, kudos to my colleague [Alexey Kalmakov](https://github.com/rndd).

```bash
#!/usr/bin/env bash

if [[ ! -f $TEAMCITY_BUILD_PROPERTIES_FILE ]]; then
  echo "TeamCity properties file could not be found. Not running within TeamCity?"
  exit 1
fi

while IFS='=' read -r key value; do
    [[ -n $key ]] && printf -v ${key//./_} "${value%% }"
done <<< "`cat "$TEAMCITY_BUILD_PROPERTIES_FILE" | grep -v '^#'`"
```

## Oh my&hellip; what's going on here?

We're feeding contents of the TeamCity build properties file into a loop, with `grep -v '^#'` filtering out lines that begin with `#` (comments). Each line is parsed by `read` which configured to split the line by `=` ([IFS](http://www.tldp.org/LDP/abs/html/internalvariables.html#IFSREF)) into `key` and `value`. Then we're using `printf -v` to declare environment variables for non empty keys with trimmed values.

Please note that dots are replaced by underscores because no one can use dots in Bash variables. For example, system property `agent.name` becomes accessible by `$agent_name`.

## And how can I use it?

Straight forward:

```bash
#!/usr/bin/env bash
# deploy.sh

source tc-props.bash

echo "Running on $agent_name"
echo "Deploying $org_app_version to $org_deploy_server..."
# ...
```

*MAGIC.*

I suggest you to avoid using weird parameter names (like this guy `ლ(ಠ益ಠლ)`) and stay simple with default syntax. As far as you can see this is not a complete parser of JAVA properties and it's robustness is in question, but it works reasonably well with TeamCity-generated files. This script is not perfect and I welcome every positive feedback for it. See also [corresponding gist](https://gist.github.com/anton-rudeshko/66d0f424470f09fa8a60).

## Conclusion

Now you can safely decouple your Bash scripts from TeamCity almighty textarea and place them into proper git repo.

I hope it will help you someday. Have a nice day!
