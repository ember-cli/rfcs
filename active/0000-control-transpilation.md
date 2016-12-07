- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Ember CLI Issue: (leave this empty)

# Summary


Allow consumer apps to control the transpilation config not only of application code, but also
of addon code.

# Motivation

Ember-cli as allowed users to write ES6 code and transpile it to ES5/3 since almost the beginning
of the project. It's one of those things that people take for granted.

Today that is done through `ember-cli-babel`, an addon that is in the default blueprint and
that the huge majority of apps use.

This transpilation process has some sensitive defaults that work for most people, but also allow
some use configuration for, by example, disable some transformations enabled by default or enable
some experimental ones.

What is less know is that this configuration only affects the application code. The transpilation
of code living in addons is done according to the configuration of those addons. While there is
use-cases for this, particularly addons that want to run their code though a custom babel plugin,
generally speaking the desired transpiled output is something that the end-users should control.

By example, applications that only target evergreen browsers may want to disable transpilation of
all ES6 feature since moden browsers already ship them.

This RFC wants to change this behaviour so addons by default honor the host app's preferences,
while allowing those addons to run their own transformations.

# Detailed design

The idea is to still allow addons to be in control of their own transpilation process, but
make them honour uses preferenced by default.

There is three different scenarios for addons:

#### The addon doesn't care about the transpilation process.
This is the most common common situation. Most addons are authored in ES6 and they don't
mess with the host app's configuration.
Those addons should just transpile their code user the host's apps settings.

#### Addons that require experimental features
There is two options here. Either addons can unconditionally enable that feature only for their
tree, or it's the user responsability to enable that feature in order to use the addon, with
optionally the addon doing some feature detection to warn (not force) the user to enable that feature.

By example, an addon that uses `async-await` which right now are only available in Chrome Canary
could either use the host app's configuration plus this feature or it could just assume the user
has enabled it and tell them to do so in the docs. Optionally the addon could do feature detection
(easened by some utilities in `ember-cli-babel` that abstract presets and all that configuration)
to show a warning in the console.

I believe that the user should have the last word on this, and given the small amount of addons
that actually do this seems that it's reasonable to update them to behave that way.

#### Addons that add their custom babel plugins

The number of addons that does this is even smaller, but one of them is nothing less than Ember Data,
which uses a babel-plugin to strip Heimdall's instrumentation from production builds.

Since addons remain in control of their transpilation and just reuse the app's configuration, addons
can attach their own plugins.

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
