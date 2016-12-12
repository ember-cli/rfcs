- Start Date: 2016-12-07
- RFC PR: (leave this empty)
- Ember CLI Issue: (leave this empty)

# Summary

Change how addon code is transpiled to it honors user preferences by default, but still
allow addons to be in control of the transpilation if they need to.

# Motivation

Ember CLI has allowed users to write ES6 code and transpile it to ES5/3 since almost the beginning
of the project. It's one of those things that people take for granted.

Today that is done through `ember-cli-babel`, an addon that is in the default blueprint and
that the huge majority of apps use.

This transpilation process has some sensible defaults that work for most people, but also allow
some use configuration for, by example, disable some transformations enabled by default or enable
some experimental ones.

What is less known is that this configuration only affects the application code (the one in your app
and the code in the `/app` folder of addons).
The transpilation of code living in the `/addon` folder of addons is done according to the
configuration of those addons, which is the default config of ember-cli-babel unless the addons specifies
otherwise.

The reasoning behind that decission is that addons might have special needs so they have to be in
charge their own transpilation. Some examples of addons that do that are [Ember-data](https://github.com/emberjs/data/blob/master/index.js#L115-L125),
[smoke-and-mirrors](https://github.com/runspired/smoke-and-mirrors/blob/master/index.js#L27-L42).

However, an undesired side effect of this approach is that, as it stands today, it means that
the configuration of the user is ignored by addons. That means addons will still transpile features
(p.e. generator functions) even if the users have disabled them in their apps because they targets
only evergreen browsers.

With ES6 being fully implemented in all modern browsers and many ES2016/ES2017 already starting to
be appear, seems clear that the transpilation level of apps is a moving target and addons should
obey user configuration by default, as opposed of the current approach of transpiling anything that
is not ES5 despite of what the user wants.

This RFC wants to change this behaviour so addons by default honor the host app's preferences,
while allowing those addons to run their own transformations.

# Detailed design

The idea is to still have addons control their own transpilation process, but
make them honor the user's preferences by default.

The implementation is relatively straigtforward. Every addon receives a deep clone of the application's
configuration (available by example on `this.config.babel`), and `ember-cli-babel` must transpile
the addon using those options, which happen to be the options used in the container app.

Note that addons are still in control of the transpilation, they just happen to reuse the app's config
unless the addon author decides otherwise.

Addons that require custom plugins can still get the configuration and modify it, but since it is
a deep clone of the app's configuration, they can be sure that any change they make will be scoped
to that addon and will not affect other addons or the container app.

Note that addon authors _could_ enable optional plugins, like per example [**decorators**](https://github.com/martndemus/ember-font-awesome/blob/master/index.js#L13-L20).
However, since experimental features might become part of the ecmascript spec, this should be
generally discouraged.

# How We Teach This

This has no impact for apps, just for addons.

Most addons will not require any change. Only those that want to modify this the transpilation
process like [ember-computed-decorators](https://github.com/rwjblue/ember-computed-decorators/blob/master/ember-cli-build.js)
or ember data will have to be updated.

# Drawbacks

Although this change *does not* require upgrading to BabelJS, it will be something to take into account
by whatever transition happens last.

# Alternatives

None that I can think of, beside of not doing this.

# Unresolved questions

For this approach it doesn't matter if the version of babel is 5 or 6, however the plugins and their
names are funamentally incompatible.

- Should addons that depend on `ember-cli-babel` be compiled with their version or with the app's version?
- If the version of babel in the addon wins but the host apps uses babel6, the config cannot be shared. In
  that case addons should get the default config?
