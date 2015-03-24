- Start Date: 2015-03-24
- RFC PR: 
- ember-cli Issue: 

# Summary

Enabling addons to specify custom prompts that would inquire the user, and perform arbitrary actions based on those responses. Actions could range from running blueprints, installing additional addons, setting configuration.

Similar to what Yeoman has:

![from yeoman](http://yeoman.io/assets/img/codelab/image_8.5a17.png)

# Motivation

This would pave the way for users creating custom 'app' blueprints that can have their personal/comapany's workflows, with default settings, and optionals. This could include installing the addons you always install, e.g. ember-cli-less, bootstrap, font-awesome, etc.

This would also make addons much more powerful, since they can ask the user critical questions, and not always assume. This would also make ember-cli more powerful, and as we draw closer to a core cli, these features will be more important for wider adoption.

# Detailed design

**TODO**

Add a `prompts` hook for addons, which would utilize `ui.prompt` behind the scenes.

One issues that prompts introduce is when you aren't interacting with your setup i.e. CI or testing, and also the possibility to overwhelm a user. Having a `--skip-prompts` option would mitigate issues like that, and prompts should have defaults, so skipping prompts would just do the default.

# Drawbacks

* More API surface area (more to test, and fix).


# Alternatives

None.

# Unresolved questions

How to do this.
