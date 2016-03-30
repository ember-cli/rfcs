- Start Date: 2015-03-24
- RFC PR:
- ember-cli Issue:

# Summary

Enabling addons to specify custom prompts that would inquire the user, and perform arbitrary actions based on those responses. Actions could range from running blueprints, installing additional addons, setting configuration.

Similar to what Yeoman has:

![from yeoman](http://yeoman.io/static/image_4.d4f761cf89.png)

# Motivation

This would pave the way for users creating custom 'app' blueprints that can have their personal/comapany's workflows, with default settings, and optionals. This could include installing the addons you always install, e.g. ember-cli-less, bootstrap, font-awesome, etc.

This would also make addons much more powerful, since they can ask the user critical questions, and not always assume. This would also make ember-cli more powerful, and as we draw closer to a core cli, these features will be more important for wider adoption.

# Detailed design

Prompts are defined at the blueprint level, so you can create
a prompt that runs during `ember install my-addon` or during the explicit `ember g my-blueprint`.

Prompts are defined using an additional object defined on the objects in the `availableOptions` array. here are default values that map to the parent object's values, but everything can be overwritten to explicitly set or override
the options defined by default.

```js
availableOptions: [{
  name: 'local',
  type: Boolean,
  aliases: [ 'l' ],
  default: false,
  description: 'whether release commit and tags are locally or not (not pushed to a remote)',
  validInConfig: true,
  prompt: {
    //name - defaults to camelized name above
    //type - defaults to logical mapping from the same field above
    //default - defaults to the value above
    //message - defaults to the description above
    //..any other options from inquirer.js
  }
}]
```

A new CLI flag is going to be added, called `--skip-prompts` which basically doesn't run the prompts portion of the available options. This flag, along with the `this.ui.ci` are used to skip the prompts explicitly or implicitly if running the addon
in a CI environment, like Travis. This allows for a graceful fallback, and also a way to not break user's tests or require
user feedback for whatever reason, like during debugging.

An addon developer doesn't need to define all of the inquirer.js prompt options, since we use some defaults
from the parent attributes, like `name`, `description` and `type`. Although, we still allow for fine-grained configuration
if the developer wants to override some option that doesn't match with their requirements. The default options look something like this:

```js
{
  name: inflection.camelize(item.name, true),
  type: self._defaultPromptType(item),
  message: item.description,
  default: item.default
}
```

The prompts are called automatically by Ember CLI just before the `locals` hook, and return the merged options
as the argument to `locals`.


# Drawbacks

* More API surface area (more to test, and fix).


# Alternatives

None.

# Unresolved questions

* `ember install` can take a list of addons. This creates a problem of possible collisions for CL arguments mapping to prompts. Possible solution is to pass all args and hope for the best, or the alternative would be to allow for a prefix.
