- Start Date: 2016-04-04
- RFC PR: #47

# Summary

One of the nice things about Node.js being our server environment is that we are not beholden to the vast installed base of browsers and can instead move our community by fiat. However, we've never explicitly communicated what our plans for support will be and our current pattern has grown organically. We need to define these guidelines so that larger organizations are able to plan for future migrations.

# Motivation

We've avoided adding libraries like Babel into our ember-cli applications in order to maintain debuggability and reduce magic. This leaves us yearning for the features, flexibility, and syntactic sugar of more-recent V8 releases. In order to not be disruptive we have continued to support the 0.10 branch which has tied us to an ancient V8 engine.

Additional issues with old versions of Node.js not related to language features:
* Bundling old and questionably working versions of `npm`.
* Bundling ancient and entirely unsupported versions of libuv and v8.
* Missing some core API and performance features such as `execSync` or `vm` on Node.js 0.10.

# Proposal

Commits to the `HEAD` of the master branch will provide support for any Active (not Maintenance Mode) Node.js LTS and the current stable Node.js version(s). This will be enforced via CI, preventing us from landing code which won't work with versions which we support. This means that our schedule for support is tied to the [LTS release schedule for Node.js](https://github.com/nodejs/LTS#lts_schedule).

For our currently existing support of 0.10 and 0.12 we should drop support at the end of their Maintenance Mode–their official end-of-life from Node.js. This gives time for our community to migrate off of those environments which we presently support.

## Legacy Support

* v0.10: LTS Maintenance Mode ends on 2016-10-01. We drop all support on that date.
* v0.12: LTS Maintenance Mode ends on 2017-04-01. We drop all support on that date.

## Ongoing support per this RFC:

These examples assume major version bumps. This is not required per Node.js policy in order to become an LTS.

* v4.2: LTS Maintenance Mode begins on 2017-04-01. We drop support on that date for new commits to `master`. _We must have an Ember CLI LTS strategy specified by this date._
* v5: Stable release, never LTS. Began support 2015-10-01. All support expires on 2016-07-01.
* v6: Released as Stable approximately 2016-04-01, converted to LTS Active Support 2016-10-01. Begin support on 2016-04-01. Active support ends on 2018-04-01 at which point we drop support on that date for new commits to `master`.
* v7: Stable release, never LTS. Begin support upon release 2016-10-01. All support expires on 2017-07-01.

# Release Process and Support Policy

The Ember CLI release process and support policy is presently `undefined`. We are currently specifying a release process in RFC #46. We must specify a support policy no later than 2017-04-01. The support policy doesn't have to be complete before then as we will continue to 100% support all Node.js supported releases until that date under this RFC.

This RFC is distinct from providing guarantees that some version of Ember CLI will run securely on each Node.js LTS release. Currently we provide no guarantees of support for any previously released version of Ember CLI–which is a gap we wish to correct and are working toward.


# Drawbacks

This will continue to delay adoption of new and exciting features, such as `async` and `await`, which are progressing through stages of standardization. However, it also makes it explicitly clear when we can expect to begin supporting certain features. It will also make it more difficult to support security fixes which need to be cherry-picked to older release branches that support older versions of Node.js but we hope to write perfect code and never need to release patches to previous versions. We also expect that the security patches that would require this to be infrequent.

# Alternatives

We could exactly match the Node.js LTS lifecycle. This sets very clear expectations but doesn't give us much freedom to evolve our usage of modern JavaScript features and could possibly force us to maintain our own forks of portions of the JavaScript ecosystem as we would assuredly have one of the longest support policies on record.

# Artifacts

We'll describe this resolution in the README and the documentation for Ember CLI for users.
