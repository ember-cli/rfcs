- Start Date: 2016-07-15
- RFC PR:
- ember-cli Issue:

# Summary

Make in-repo-addons as self-contained as full addons.

# Motivation

An in-repo addon is a great way to isolate code in an application without
adding the extra costs associated with version management. In some cases,
moving code from `app/` to `lib/my-addon/` is the first step towards
releasing it as a full, open-source addon.

Unfortunately, there are two important ways in which in-repo addons fail
to meet the goal of isolation:

 1. They cannot declare NPM dependencies
 1. They cannot declare tests

# Detailed design

### NPM Dependencies

An in-repo addon has exactly the build-time `devDependencies` and runtime
`dependencies` as the parent application. This limits the tool's
applicability for use-cases like the following:
 * add a dependency e.g., [ember-cli-content-security-policy](https://github.com/rwjblue/ember-cli-content-security-policy)) and configure it
 * add a dependency (e.g. [ember-tooltips](https://github.com/sir-dunxalot/ember-tooltips)) and some custom CSS for it;
 * add a dependency (e.g. [ember-modal-dialog](https://github.com/yapplabs/ember-modal-dialog)) and create a component that simplifies the most
   common cases (e.g. `lib/modals/app/my-simple-modal/component.js`)
 * add a depdency and patch it to fix a bug

The desired behavior is that members of `package.json`'s `ember-addon.paths`
are treated like other dependencies. In particular, their dependencies should
end up in `node_modules/`. If a dependency is an ember addon,
its `app/` and `addon/` folders are made available to the application
just as if they had been declared as direct dependencies.

The global `npm install` command doesn't know about this logic. Only
and ember-cli command could know about this structure. This RFC proposes
three more-or-less equivalent options:

 1. `ember install` with no arguments
 1. `ember install` with a flag such as `--npm`
 1. `ember npm-install`

The command should ensure that the dependencies from the in-repo addons and
the parent app are mutually satisfiable. That is, it should fail if one
in-repo addon declares `"ember-foo": "1"` and another `"ember-foo": "2"`.

See also [ember-cli/ember-cli#5016](https://github.com/ember-cli/ember-cli/issues/5016)

### Bower Dependencies

In-repo addons aren't ever "installed", which means that they don't
automatically run the blueprint with the same name as the addon.
However, given

```js
// lib/my-addon/blueprints/my-addon/index.js:
module.exports = {
  normalizeEntityName: function() {},

  afterInstall: function() {
    return this.addBowerPackageToProject('x-button');
  }
};
```

`ember g my-addon` does cause the `afterInstall` hook to run. This behavior
mimics that of an already-installed external addon.

### Tests

Adding a test file to `lib/my-addon/tests/*-test.js` does not cause it to be
run with the parent app's test suite. Nor is there a way to run just the
tests for one addon.

One major question is _how do these tests run_?

#### Injected

One option is to simply include them in the tree for the parent app's test
suite.

 * *Pro*: existing workflows and scripts will include new tests automatically
 * *Con*: provides less isolation than external addons have

`ember test` already has filtering capabilities, so it is possible to run
only the tests from a single addon. It is slightly more difficult to run
only the app's tests -- at least without something like
[ember-exam](https://github.com/trentmwillis/ember-exam).

#### Isolated

A second option is to require each addon to have its own `dummy/` app,
just like an external addon.

 * *Pro*: most documentation for external addon development will apply
   to in-repo addons
 * *Pro*: upgrading an in-repo addon to an external, open-source addon
   is trivial
 * *Con*: existing workflows and scripts will have to be modified to
   run _all_ of the app's tests (footgun)
 * *Con*: full test runs will require several build phases, slowing them down
 * *Con*: the `dummy/` apps could get out-of-sync with the main app,
   making it more difficult to upgrade core libraries like `ember`
 * *Con*: if two in-repo addons are to each rely on a third, there must
   be a way for them to declare that dependency, and ember-cli tools
   must understand it. The `npm install` wrapper's logic could become
   quite complicated.

To run the tests for an in-repo addon, change to the addon's directory
and run `ember test`, just like for an external addon.

#### App Author's Choice

It may be that some addons are truly specific to the app and are simply a
means of increasing cohesion. The simplicity of the injected approach might
win for these addons. Other addons could be on the path to open-sourcing;
for these, the better parity with external addon structure of the isolated
testing approach might win. It should be possible to support both cases
on a per-in-repo-addon basis, possibly via a flag in `index.js`.

#### Summary Recommendation

If _injected_ tests are easier to make work, support them at first. Over
time, if _isolated_ tests are required, allow in-repo addons to opt in
to them.

See also [ember-cli/ember-cli#4461](https://github.com/ember-cli/ember-cli/issues/4461)

# Drawbacks

 * it could slow down the build or rebuild phases
 * a dependency that does not appear in `package.json`'s `dependencies`
   will appear in the app, which could further confuse an already confusing
   dependency-management situation

# Alternatives

Unknown

# Unresolved questions

 * does an in-repo addon have an `afterInstall` hook? When would it run?
 * are the tests for in-repo addons injected into the parent app's test suite?
