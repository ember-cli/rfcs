- Start Date: 2016-12-07
- RFC PR: (leave this empty)
- Ember CLI Issue: (leave this empty)

# Summary

Rely on `npm` and `bower` being installed globally instead of making them
costly dependencies of `ember-cli`.


# Motivation

We are currently specifying [`npm`](https://github.com/ember-cli/ember-cli/blob/v2.10.0/package.json#L99)
and [`bower`](https://github.com/ember-cli/ember-cli/blob/v2.10.0/package.json#L40)
as dependencies of `ember-cli`, which means that every project using
`ember-cli` will currently have a copy of those tools installed in
their `node_modules` folder.

The motivation for originally bundling them as dependencies was being able
to bring a consistent experience to all `ember-cli` users and working around
known bugs in certain versions of `npm`.

Analysis has shown that those two dependencies are the cause for at least 50%
of all subdependencies, wasted install time and the disk space in the
`node_modules` folder. This makes it obvious that it is something we should
try to avoid if possible.

While initially `npm` and `bower` have been somewhat unstable, things have
improved lately and at least the common `install` subcommand should be
considered known territory.

It should be noted that all functionality we currently use of both
dependencies are:

- `npm install`
- `npm uninstall`
- `bower install`

Those features are probably the most commonly used subcommands and as such
the hopefully best tested ones.

Another advantage of this approach would be that supporting
[`yarn`](https://yarnpkg.com/) will become much easier as well since we
could transparently fallback to `npm` in case `yarn` is not available.
We also wouldn't have to package `yarn` as a dependency either, next to
`npm` and `bower`.

Yet another advantage is that dropping or deprecating support for `bower`
in `ember-cli` will be as easy as just not invoking the globally installed
`bower` executable.


## TL;DR

- faster `npm install` time for all `ember-cli` users
- faster CI builds due to that
- smaller `node_modules` folder
- less overwhelming dependency tree
- faster internal `ember-cli` tests
  (since less content in `node_modules` needs to be copied)
- easier support for `yarn` with `npm` fallback
- simpler deprecation for `bower` support


# Detailed design

[#6503](https://github.com/ember-cli/ember-cli/pull/6503) has already started
moving from JS API invocation of NPM to command line execution via
[`execa`](https://github.com/sindresorhus/execa). This is a requirement for
using the global NPM since using the JS API is 1) discouraged and 2) might
not be stable across releases, while the `install` command hasn't changed
in a significant way.

The same thing will need to happen for `bower`, even though their interactivity
during install (e.g. asking for version conflict resolution) will make this
a little bit more difficult.

Once both package managers are invoked this way we can use e.g.
[`node-which`](https://github.com/npm/node-which) to figure out the path to
the globally installed `npm` and `bower` commands and run those instead.
If those tools are not found we should show an error and provide instructions
on how to install them globally.

It might be beneficial to introduce a version check for both commands before
running then via `npm --version` and `bower --version` to make sure we are
invoking a compatible version. While this should be the case for all recent
releases, we never know what `npm` might change in the future, so we should
show a warning when we notice a potentially incompatible new major release.

In case of `yarn` support we should first try to find the `yarn` executable
and if it exists we should use it to install the required dependencies. If
it doesn't exist we can fallback to `npm` and print a message on how to
install it, or if a `yarn.lock` file exist we show an error that `yarn`
could not be found.

`execa` has a `preferLocal` option, which looks for a matching executable
in the local `node_modules` folder first. This lets our users add `npm`
and `bower` to their own `package.json` files if they wish to keep using
pinned versions of those package managers.


# How We Teach This

In the view of our users nothing substantial will change. The `README.md`
files in our default blueprints already suggest to have `npm` and `bower`
installed globally so it can be considered uncommon to invoke those commands
like `node_modules/.bin/npm`. Our `.travis.yml` file installs both globally
as well, so that doesn't need any changes either.

`npm` is already distributed with every Node.js install so it is uncommon
to not have it available globally. If `bower` was not installed globally we
will show a helpful warning with the command needed to install it globally.

We should however write a blog post about this change letting the users know
that from now on their global versions are used and that they should keep
their installations of `npm` and `bower` up-to-date to make sure they don't
hit any issues.


# Drawbacks

As mentioned in the "Motivation" section we will lose the very tight control
over what `npm` and `bower` versions our users are running, while at the same
time we are no longer responsible for making sure our users have all the `npm`
bugfixes available in their local version.


# Alternatives

The obvious alternative is keeping the dependencies bundled as they are and
updating them every once in a while. The impact of this is keeping the same
long `npm install` times that we currently have. Even though `yarn` might
help bring those times down, not installing stuff will always be faster than
installing stuff fast.


# Unresolved questions

TBD
