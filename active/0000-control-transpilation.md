- Start Date: 2016-12-07
- RFC PR: (leave this empty)
- Ember CLI Issue: (leave this empty)

# Summary


Allow consumer apps to control the transpilation config not only of application code, but also
of addon code.

# Motivation

Ember CLI has allowed users to write ES6 code and transpile it to ES5/3 since almost the beginning
of the project. It's one of those things that people take for granted.

Today that is done through `ember-cli-babel`, an addon that is in the default blueprint and
that the huge majority of apps use.

This transpilation process has some sensible defaults that work for most people, but also allow
some use configuration for, by example, disable some transformations enabled by default or enable
some experimental ones.

What is less known is that this configuration only affects the application code. The transpilation
of code living in addons is done according to the configuration of those addons. While there is
use-cases for this, particularly addons that want to run their code though a custom babel plugin,
generally speaking the desired transpiled output is something that the end-users should control.

By example, applications that only target evergreen browsers may want to disable transpilation of
all ES6 feature since moden browsers already ship them.

This RFC wants to change this behaviour so addons by default honor the host app's preferences,
while allowing those addons to run their own transformations.

# Detailed design

The idea is to still have addons control their own transpilation process, but
make them honour user preferences by default.

There are three different scenarios for addons:

#### The addon doesn't care about the transpilation process.
This is the most common common situation. Most addons are authored in ES6 and they don't
mess with the host app's configuration.
Those addons should just transpile their code user the host's apps settings.

#### Addons that require experimental features
There are two options here. Either addons can unconditionally enable that feature only for their
tree, or it's the user responsability to enable that feature in order to use the addon, with
optionally the addon doing some feature detection to warn (not force) the user to enable that feature.

By example, an addon that uses `async-await` which right now are only available in Chrome Canary
could either use the host app's configuration plus this feature or it could just assume the user
has enabled it and tell them to do so in the docs.

#### Addons that add their custom babel plugins

The number of addons that does this is even smaller, but one of them is nothing less than Ember Data,
which uses a babel-plugin to strip Heimdall's instrumentation from production builds.

Since addons remain in control of their transpilation and just reuse the app's configuration, addons
can attach their own plugins.

#### How can addons be in charge of their own transpilation and yet honor user preferences?

Ember-cli should provide to each addon a deep clone of the configuration defined in the host app
(by example available in `this.options.babel`), and that config is the one used to transpile the addon.

The addon will therefore honor the app's configuration by default, but it can also decide to add
custom plugins, enable specific features or just throw a warning message if the app's config
doesn't have required specific transformation enabled.

By modifying this clone, their changes will not affect the host app or other addons.

# How We Teach This

This has no impact for apps, just for addons.

Most addons will not require any change. Only those that want to modify this the transpilation
process like [ember-computed-decorators](https://github.com/rwjblue/ember-computed-decorators/blob/master/ember-cli-build.js)
or ember data will have to be updated.

# Drawbacks

We give the users more control over the transpilation process at the expenses of some complexity
for addon authors that want to use experimental ES2017 features in their addons, leaving
up to them to give a good user experience.

# Alternatives

None that I can think of, beside of not doing this.

# Unresolved questions

For this approach it doesn't matter if the version of babel is 5 or 6, however the plugins and their
names are funamentally incompatible.

- Should addons that depend on `ember-cli-babel` be compiled with their version or with the app's version?
- If the version of babel in the addon wins but the host apps uses babel6, the config cannot be shared. In
  that case addons should get the default config?
