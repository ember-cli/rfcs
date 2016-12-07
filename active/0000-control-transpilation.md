- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Ember CLI Issue: (leave this empty)

# Summary

Allow consumer apps to control the transpilation not only of application code, but also
of addon code.

# Motivation

At the moment addons control their own transpilation process. This is nice for having a
"just works" experience, but leaves users without control over the transpiled code they
ship.

By example, users might want to opt-out from transpiling certain features because
the application is going to be targeted to fully capable ES6 browsers is still shipping
ES5 code of their addons, regardless of the babel settings of the app. This leads to
bundles more verbose than really required.

The generator functions used by ember-concurrency is a good example of an utterly verbose
transpiled code that users might not want.

The goal of this RFC is to make EmberCLI honor the user's preferences for transpilation (through
`ember-cli-babel` or perhaps `typescript`).

# Detailed design

The proposed change in behaviour is to apply the user's preferences to the transpilation
of the `addon` tree instead of the addon's configuration. The addon's configuration is
honored by their dummy app only.
For a overwhelming majority of the addons that just use the default settings they won't
notice anything.

There might be a small number of addons that do customize the settings used by ember-cli-babel,
and there is where some action is required.

The suggested approach allow addons to access the babeljs configuration so, if they require
an experimental feature enabled to work, they can throw a warning (or even a hard failure)
if that feature is not enabled in the host's configuration.

It would be a responsability of the addon to emit those warnings if needed, so ember-cli must provide
an idiomatic way of accessing the application's configuration. At the moment it is possible
but is somewhat hacky (`_findHost().options.babel`) and is likely going to change once
Babel6 is used, which will probably use a `.babelrc` file for configuration instead of the
current approach as options in the `ember-cli-build.js` file.

# How We Teach This

For end users everything will remain the same.
For most addon authors it will not require any change. Only those addons that do unusual
things with babel, like [ember-computed-decorators](https://github.com/rwjblue/ember-computed-decorators/blob/master/ember-cli-build.js)
may want to do something about it.
If they don't do anything, things may or may not work depending on the user's settings,
so it would be sensible for them to inspect the configuration and warn if neccesary.

Checking if a feature is available is probably not trivial, so `ember-cli-babel` should
expose some helpers that can be required from node-land to verify that the hosts app supports
a given feature.
This can also abstract babel5/6 complexities, presents and such.

# Drawbacks

We give the users more control over the transpilation process at the expenses of some complexity
for addon authors that want to use experimental ES2017 features in their addons, leaving
up to them to give a good user experience.

# Alternatives

What other designs have been considered? What is the impact of not doing this?

# Unresolved questions

Optional, but suggested for first drafts. What parts of the design are still
TBD?
