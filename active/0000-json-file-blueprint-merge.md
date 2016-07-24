- Start Date: 2016-07-24
- RFC PR:
- ember-cli Issue:

# Summary

This feature would allow blueprints and ember upgrades to interactively merge JSON files instead of wiping away existing files, or causing syntax errors due to manual text editing.

# Motivation

The current process for including things like `bower.json`, `package.json`, `.babelrc`, etc leads the user to have to manually read through diff logs and resolve change issues.
Most often, this occurs with the `ember-cli` app upgrade blueprint.
Especially for newer developers, it can be confusing why some of your addons get uninstalled in the process of upgrading.

# Detailed design

Since both the existing and suggested JSON files can be parsed and read as JSON, it makes sense that we could diff each key and value programmatically and have more information than a simple text comparison.
Using [json-merge-interactive](https://github.com/rtablada/json-merge-interactive) we could ask the user for each conflicting key (or even use semver comparisons to make educated guesses) and guide them along the way.

# Drawbacks

* This design does not account for removed packages, addons, or JSON nodes.
* This will lead to resorting of JSON file key value structures because of the nature of `JSON.stringify`.

# Alternatives

Another way around this could be to use a descriptive broccoli style DSL to allow changes to common files such as `package.json` and `bower.json` (especially for upgrades).

# Unresolved questions

* Does the proposed [json-merge-interactive package](https://github.com/rtablada/json-merge-interactive) give enough control for file diffing?
* Are there needed improvements in performance to the json-merge-interactive.
* How do we tell when files should use the traditional merge and diff vs JSON diff (file extension, registration in the blueprint, etc)?
