- Start Date: (fill me in with today's date, 2016-08-06)
- RFC PR: 
- ember-cli Issue: 

# Summary

The new asset import API, available since ember-cli 2.7.0, offers the possibility to 
import Ember CLI addons from Ember CLI addons (aka, nested addons).
This RFC intends to find solutions for typical edge cases.

# Motivation

Since ember-cli 2.7.0, addons can be nested if the nesting floows the following
scenario:
* The nested addons do not make use of the property `parent.bowerProperty` to define
  assets paths because it not always defined (it is not defined in EmberAddon instances)
* The nested addons do not need to execute a default blueprint.

Solving these use cases would allow for a simpler, more generic, addons development
process, and will allow teams to build more reusable and higher level addons for
their projects and the community (like [ember-admin](https://github.com/DockYard/ember-admin)
using [ember-power-select](ember-power-select.com) with no additional boilerplate, for instance).

# Detailed design

Here is a rundown of the questions raised with @nathanhammond in [a previous PR](https://github.com/ember-cli/ember-cli/pull/6111):

1. Should nested addons be allowed to use their grandparents' `bower_components` directory?
1. How/When should the default blueprints of the nested be run?
1. How to deal with conflicting versions (The app uses my-addon 1.0.0, but my-addon 2.0.0 is required as a nested component)?

For each of these questions, here's a first attempt towards an answer:

1. Yes, they should, because there's only one bower directory available to the app, and it should be available to the whole app,
  including to the dependencies. See [the PR linked above](https://github.com/ember-cli/ember-cli/pull/6111) for an example.
1. They could be run in the "afterInstall" hook, when the dependencies (hence nested addons) are already installed.
  See [this comment](https://github.com/ember-cli/ember-cli/pull/6111#issuecomment-235829717) for a specific implementation
1. Ideally, this falls under Bower's (or npm's) umbrella.

The same way that ember-cli hides the complexity for installing addons and the related node deps and bower deps,
the initial idea is to hide the complexity to do (most of) the hard work in ember-cli itself.

# Drawbacks

We should be extra careful with this, as we are likely to fall in one of the
many traps that package managers have to face, dealing with conflicts, resolutions, ordering, tree buiding, etc...
The more we can rely on the package managers, the less code to embark in ember-cli, the better.

# Alternatives

Instead of hiding the complexity and have ember-cli orchestrate the installation of the complex addons by itself,
exposing new APIs to let the developer do as they must for the nested addons would let us see the use cases and find
opportunities to simplify later.

Following that, the alternative could be to:
* make the `EmberApp#_findHost` API public, and proably rename it. (answers the `bowerDirectory` problem with: `this.findHost().bowerDirectory`)
* expose in the EmberAddon instance the addons it depends from.
* expose a new API to execute blueprints (answers the blueprint execution problem with the above)
* Let Bower handle conflicting versions (answers the versions' conflict resolution)

# Unresolved questions

Many. The document will be updated as they emerge.
