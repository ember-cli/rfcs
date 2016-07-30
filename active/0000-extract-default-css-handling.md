- Start Date: 2016-07-30
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Extract the built-in CSS handling inside of Ember CLI to an addon (`ember-cli-css`) which registers itself identically to the way `ember-cli-sass` or `ember-cli-less` functions.

# Motivation

The default handling of CSS inside of Ember CLI is ad hoc and generally underspecified. It functions as a primitive CSS preprocessor but doesn't register itself as such. There are a lot of issues that we experience as a consequence of this approach:
- [We literally have questions in the documentation.](https://github.com/ember-cli/ember-cli.github.io/blame/6205e1f73da9a4edceeb5be70a93ef058b223975/_posts/2013-04-09-asset-compilation.md#L130)
- The outcome of any particular CSS setup is left to the imagination and reverse-engineering of the user–approximately programming by permutation.
- There is no way to configure what happens inside of addons without manually writing a `treeForStyle` hook.
- [Behavior of underlying packages](https://github.com/ember-cli/ember-cli/pull/5874) can break builds which can force us to release new versions of Ember CLI.
- The behavior of CSS handling is unnecessarily tied to the Ember CLI version and it makes the process of landing improvements to our CSS handling much more difficult than truly necessary.

Moving it to an addon allows us to:
- Iterate faster.
- Not couple the behavior to Ember CLI.
- Provide understandable guarantees about what happens inside of addons when they include `ember-cli-css`.
- Make it more clear that this is something which can be swapped out for something like `ember-cli-sass`.

# Detailed Design

The first step in the process is to simply extract all of the default CSS handling out into an addon which implements the exact same behavior as we presently have. For the first release there will be _no_ changes in the outcome of the build. This will become part of the default Ember CLI blueprint. This allows us to meet our backwards compatibility guarantees.

After the extraction we will then be able to allow our users to select which version of the `ember-cli-css` addon they wish to use which allows us to start moving it toward reasonable defaults for application- and addon-level behavior.

# How We Teach This

The primary change will be to our documentation at [ember-cli.com](https://ember-cli.com/user-guide/#css). We'll need to make it more clear that you must swap the default-included `ember-cli-css` behavior for any of the other preprocessors, and that the default CSS version functions identically to a preprocessor.

Further we should use this opportunity to add additional documentation about the inclusion of preprocessors inside of addons and how that functionality works.

# Drawbacks

When users first upgrade to an Ember CLI version that doesn't include the CSS behavior by default they will be _required_ to add the addon to their `package.json`. As most people use more advanced preprocessors this is likely to be a small audience.

# Alternatives

No change.

# Unresolved questions

- How much change do we want to allow within the addon and is that in keeping with our SemVer guarantees?
- Do any of the existing addons rely on the default CSS behavior in order to function?