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

Prompts are defined using `ui.prompt`, which pulls data from the blueprint's `availableOptions` if it has `argName` defined or there are options available. For example:

```js
availableOptions: [{
  name: 'local',
  type: Boolean,
  aliases: [ 'l' ],
  default: false,
  description: 'whether release commit and tags are locally or not (not pushed to a remote)',
  validInConfig: true
}]
```

Which can be utilized by `ui.prompt` automatically as long as the author does something like:

```js
return this.ui.prompt([
  {
    name: 'useLocalGit',
    type: 'confirm',
    values: ...,
    argName: 'local' // this maps to the option `name` above
  }
  // more prompts
]);
```

This way the prompt can pull in default values or values specified via the CL, e.g. `--local`. Default values would be used if `--skip-prompts` is used. Options defined in `availableOptions` can be used as default values in certain scenarios, like if the user didn't specify a `prompt.type`, then the argument `type` could be inferred from, e.g. `Boolean` => `'confirm'`. This could also go for reusing the description if one isn't provided in the prompt.

As a fail safe, addon authors will have a warning or an error if there are no default values for their prompts. This ensures that addons can be used in tests and CI systems, as well as for user convenience.

Prompts can be specified in `beforeInstall`, `beforeUninstall`, `locals`, `afterInstall` and `afterUninstall`. It should always be used in the promise form, i.e. no callback.

## Advanced

For advanced use-cases the addon author can dig into the prompt results and tweak using the returned promise.

```js
return this.ui.prompt([
  // prompts
]).then(function (answers) {
  // tweak the answers
  return answers;
})
```

# Drawbacks

* More API surface area (more to test, and fix).


# Alternatives

None.

# Unresolved questions

* `ember install` can take a list of addons. This creates a problem of possible collisions for CL arguments mapping to prompts. Possible solution is to pass all args and hope for the best, or the alternative would be to allow for a prefix.
