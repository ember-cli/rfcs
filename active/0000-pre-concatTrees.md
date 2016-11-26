- Start Date: 2016-11-26
- RFC PR: (leave this empty)
- Ember CLI Issue: (leave this empty)

# Summary

Build hook for getting all trees prior to concat into their final output
files.

# Motivation

Moving certain logic to compile-time can yield significant perf wins.
However, ember-cli does not currently provide a build hook that allows
addons to get a full project view directly prior to the final concat
step. Having this pre-concat tree from `applicationJs` would allow
developers and addons to hook in directly prior to the final phase and
make informed actions upon the project as a whole.

# Detailed design

We can simply add a new hook within `EmberApp.prototype.javascript` and
`EmberApp.prototype.styles`. Something akin to:

```js
// psuedo-code, obviously :p
Ember.prototype.preConcatTrees(type, tree) {
  return addons.reduce((addon) {
    return addon.preConcatTree(type, tree);
  }, this.addons);
}
```

Then called with:

```js
applicationJs = this.preConcatTrees('js', applicationJs);
```

# How We Teach This

This would be documented as another available hook along with the
motivation and general purpose.

# Drawbacks

Adding an additional hook that interates over all addons could increase
build time.

# Alternatives

Perhaps this functionality already exists but I missed it.

# Unresolved questions

Should this be limited to just addons or also app space?
