- Start Date: 2016-04-04
- RFC PR: #47

# Summary

One of the nice things about Node.js being our server environment is that we are not beholden to the vast installed base of browsers and can instead move our community by fiat. However, we've never explicitly communicated what our plans for support will be and our current pattern has grown organically. We need to define these guidelines so that larger organizations are able to plan for future migrations.

# Motivation

We've avoided adding libraries like Babel into our ember-cli applications in order to maintain debuggability and reduce magic. This leaves us yearning for the features, flexibility, and syntactic sugar of more-recent V8 releases. In order to not be disruptive we have continued to support the 0.10 branch which has tied us to an ancient V8 engine.

# Proposal

We track the [existing LTS releases for Node.js](https://github.com/nodejs/LTS#lts_schedule), but only provide support for the active LTS releases, or the stable versions which will become LTS releases. Once an LTS is moved into maintenance mode we will drop support for that version. Versions of Ember CLI whose support crosses one of the Node.js boundaries will be supported per the ember-cli release support policy which is currently unspecified, though described by RFC #46.

For our currently existing support of 0.10 and 0.12 we should drop support 2016-10-01 and 2017-04-01, respectively, at the official end of support for those releases. This gives time for our community to migrate off of those environments.

# Drawbacks

This will continue to delay adoption of new and exciting features, such as `async` and `await`, which are progressing through stages of standardization. However, it also makes it explicitly clear when we can expect to begin supporting certain features. It will also make it more difficult to support changes which need to be cherry-picked to release branches that support older versions of Node.js but we hope to write perfect code and never need to release patches to previous versions.

# Alternatives

We could exactly match the Node.js LTS lifecycle. This sets very clear expectations but doesn't give us much freedom to evolve our usage of modern JavaScript features and could possibly force us to maintain our own forks of portions of the JavaScript ecosystem as we would assuredly have one of the longest support policies on record.

# Artifacts

We'll describe this resolution in the README and the documentation for Ember CLI for users.
