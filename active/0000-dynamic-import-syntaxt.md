- Start Date: (fill me in with today's date, 2017-02-27)
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

This RFC proposes implementing a dynamic import syntax transformation in Ember CLI.

# Motivation

Recently, the proposal for adding a "function-like" import() module loading syntactic form to JavaScript has
reached stage 3 of the TC39 process. Other asset pipeline tools like
Webpack [already support the `import()`](https://github.com/airbnb/babel-plugin-dynamic-import-webpack) syntax.

For Ember users it can be quite useful to dynamically import modules. This is well explained in the proposal.

> This could be because of factors only known at runtime (such as the user's language), for performance reasons
(not loading code until it is likely to be used), or for robustness reasons (surviving failure to load a non-critical
module). Such dynamic code-loading has a long history, especially on the web, but also in Node.js (to delay startup
costs). The existing import syntax does not support such use cases.
>
>Truly dynamic code loading also enables advanced scenarios, such as racing multiple modules against each other and choosing the first to successfully load.

# Implementation

Once Ember CLI [supports Babel 6](https://github.com/ember-cli/ember-cli/issues/5015), it should be quite easy to
implement this RFC By using the [Babel plugin to transpile `import()` to `require`](https://github.com/genkgo/babel-plugin-dynamic-import-amd).
