- Start Date: 2015-12-21
- RFC PR: [#30](https://github.com/ember-cli/rfcs/pull/30)
- ember-cli Issue: (leave this empty)

# Summary

Rework the `app-boot` hook to make it more module-friendly, reduce the API surface area, and make it simpler to unravel the Ember application programatically.

# Motivation

Treating `app-boot` as an appended section of imperative code inside of app.js makes it difficult to reason about in a modules-only world. We only have a few occurrences of this in our extended codebases (I audited every occurrence on GitHub), which implies that we can reduce the API surface area and accomplish the same underlying goal of triggering the initial `require` that unravels everything.

* [ember-cli](https://github.com/ember-cli/ember-cli/blob/a2a9a886e146ce7bdfbdd5338542538ea7b38d14/lib/broccoli/ember-app.js#L1600) - Generates the initial require.
* [ember-cli-delay-app-boot](https://github.com/rwjblue/ember-cli-delay-app-boot/blob/master/index.js#L31) - Throws a timeout around the initial require and includes some copypasta from above.
* [ember-cli-fastboot](https://github.com/tildeio/ember-cli-fastboot/blob/2058dd1ebf08be818851d1e65651db49cdd0e69f/index.js#L73) - This adds a new specially-named module and doesn't call `require`. Instead, it itself becomes the module which must be required to boot the app.
- In the future (read: I'm already encountering this) a Service Worker may wish to tell the application when it should boot or bind that functionality onto a promise chain. My current approach is to delay responding to the app-boot request and maintain that promise chain inside of the service worker.

# Detailed design

The primary use case for `app-boot` is to tweak and modify how the initial `require` is called. None of these above use cases are dramatically different. I'd propose that we adopt the behavior of `fastboot` generically, creating a magic module that you can `require` and use to unravel the application.

As a part of doing this we would allow the application to specify the module(s) which should be required, in order, as the entry point into the application as an option in ember-cli-build. This would default to the magically generated module name but could also be configured by addons. (An empty array means `require` no modules, `undefined` means require the magic module.)

This covers all existing use cases, reduces copypasta, reduces API surface area, and makes the initial module(s) required declarative.

# Drawbacks

This will reduce flexibility, but having opinions is a good thing.

# Alternatives

Leave this alone. After https://github.com/ember-cli/ember-cli/pull/5246 lands it isn't as pressing of an issue.

# Unresolved questions

* How do we make this play nicely for multiple apps on the same page?

# What parts of the design are still TBD?

I believe that this is a complete design.