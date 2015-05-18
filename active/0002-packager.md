- Start Date: 2015-03-01
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Allow Ember applications to resolve a dependency graph when packaging applications.

# Motivation

Currently Ember CLI is not smart on how it packages up an application. It takes trees of various types, merges and concats them. While this is alright solution if your application is small it becomes problematic when the application becomes large enough that dependencies and code paths can no longer be reasoned about.

# Scope

- Break the build up into 3 distinct phases
- Traverse a dependency graph to gain selective builds
- Provide a build solution for engines
- Remove hard dependency on Bower in favor NPM
- Remove app.import API

# Definitions

- __Builder__ - Responsible for generating an array of broccoli trees that contain the app and all the addons.
- __Pre-Packager__ - Responsible for resolving and reducing the graph to a  tree containing only the modules in the graph.
- __Packager__ - Responsible for taking the resolved tree and applying concatenation strategies on the tree.
- __Entry__ - A entry node into the graph in which you traverse down from

# Detailed Design

## Builder
The Builder is a re-thinking of the existing `EmberApp` constructor function that is used in the Brocfile today.  Unlike the existing `EmberApp` constructor, the `Builder` constructor only does the following:

- Discover the addons
- Runs the addons hooks that are pre-resolved tree
- Merges an addon's app directory with the consuming app
- Transpiles ES6 code to AMD

As part of the ES6 transpilation we drop a `dep-graph.json` into the tree of the app and each addon that supplies javascript dependencies. This is used by the Pre-Packager to resolve the graph. The result of this phase is an array of Broccoli trees containing all of the built assets in the app and addons.

At this point nothing is really different then what happens today besides dropping the `dep-graph.json` and only running the addon hooks that need to happen at the beginning of a build.

## Pre-Packager
The Pre-Packager is a graph resolver algorithm for javascript dependencies. Another way of think about the Pre-Packager is a complex [Broccoli-Filter](https://github.com/broccolijs/broccoli-filter) and [Broccoli-Merge-Trees](https://github.com/broccolijs/broccoli-merge-trees).

The Pre-Packager can resolve the following types out of the box:

- Addons
- Legacy modules from the node_modules directory (CJS)
- ES6 from the node_modules directory

While the Pre-packager comes with a sane set of defaults, custom dependency resolvers can be written to allow resolution of modules outside of node_modules.

### Dependency Resolvers

There are 2 types of dependency resolvers in the pre-packager.

1. Resolvers that start traversal from a static graph
2. Resolvers that must wait till the static graph has materialized

Essentially anything that can provide a `dep-graph.json` prior to resolution falls into number 1 and everything else falls into number 2.

### dep-graph.json

The dep-graph.json looks like the following.

```json
{
  "example-app/app.js": {
    "imports": [
      "exports",
      "ember",
      "ember-resolver",
      "ember-load-initializers",
      "example-app/config/environment"
    ]
  },
  "example-app/initializers/ember-moment.js": {
    "imports": [
      "exports",
      "ember-moment/helpers/moment",
      "ember-moment/helpers/ago",
      "ember-moment/helpers/duration",
      "ember"
    ]
  },
  "example-app/router.js": {
    "imports": [
      "exports",
      "ember",
      "example-app/config/environment"
    ]
  }
}
```

### Static Graph Resolution

Apps and addon resolution uses the `dep-graph.json` to start the traversal. By default, the Pre-Packager uses the App as the entry into the graph, but developers can supply N entries. More on this later, but for now you should think of an entry is a large functional area of an application.

#### Syncing
The first step in the resolution phase is to `syncForward` the entries files to what will be the output tree. Any non-javascript dependencies are synced forward as well.

#### Resolution Algorithm
The resolution algorithm is as follows:

1. Read in the `dep-graph.json` for an entry
2. Look at the first file in the graph and `syncForward` it's imports
3. Take the first import look up it's graph and recurse
4. Recursion continues till the static graph is resolved

### Post-Static Graph Resolution

Legacy modules and ES6 code coming from node_modules must come after. This is because we do not have a static graph prior to resolution and we look at these dependencies as opaque. During the static resolution we keep track of the types we need to resolve later.  This is denoted by a prefix on the import. If you're familiar with WebPack, Browserify, or RequireJS this is a similar approach.

To hint at the Pre-Packager to resolve a legacy commonjs module use `npm:`.

```js
import numeral from 'npm:numeral';
```

 When we attempt to resolve these types of dependencies we create an artificial "main" file containing the imports we saw and then just delegate to a tool that is designed to create bundles of these types.

For instance, `npm:` uses Browserify to create bundles of CJS modules.  This type of resolution was first done by Edward Faulkner with [Ember Browserify](https://github.com/ef4/ember-browserify) and we feel it's the best way to bridge these types of gaps.

## Packager

The Packager will be passed a single resolved tree that represents a directory structure that looks like the following:

```
 'browserified/ember-qunit/ember-qunit-legacy.js',
 'dummy-tests/dep-graph.json',
 'dummy-tests/index.html',
 'dummy-tests/index.js',
 'dummy-tests/unit/components/foo-bar-test.js',
 'dummy/app.js',
 'dummy/components/baz-bar.js',
 'dummy/components/foo-bar.js',
 'dummy/config/environment.js',
 'dummy/crossdomain.xml',
 'dummy/dep-graph.json',
 'dummy/index.html',
 'dummy/index.js',
 'dummy/pods/bizz-buzz/component.js',
 'dummy/pods/bizz-buzz/template.js',
 'dummy/pods/foo-baz/component.js',
 'dummy/pods/foo-baz/template.js',
 'dummy/robots.txt',
 'dummy/router.js',
 'dummy/styles/app.css',
 'dummy/templates/components/foo-bar.js',
 'dummy/templates/profile.js',
 'ember-qunit/dep-graph.json',
 'ember-qunit/ember-qunit.js',
 'ember/dep-graph.json',
 'ember/ember.js',
 'ember/ember/get.js'
```

At this point we are ready to start concatenating the files together.  Out of the box the resolver will have 4 concatenation strategies: 'default', 'engines', 'http2', and 'custom'.

Since the graph has already been resolved you can provide an array of strategies and the output will result in the dist containing a folder for each strategy.

## Strategy Definitions

- __default__: This is the same output that we have today; app and vendor for both js and css, along with index.html, crossdomain.xml and robots.txt. This may eventually get re-named to legacy.
- __engines__: Once we have the concept of engines, developers will more than likely want to break apart into functional sets. This will allow you to build a bundle for each engine. These are looked at as entries to the pre-packager.
- __http2__: As more servers move towards using [HTTP2](https://http2.github.io/) you actually want granular assets instead of concated blobs. This is due to the fact that you have [multiplexing](https://www.mnot.net/blog/2014/01/30/http2_expectations) in HTTP2. This allows the browser to cache bust only on the things that have changed. This may change to "none".
- __custom__: If none of the out of the box steps work for you can tell the packager that you're are going to write your own concatenation logic.

### Custom Concatenation
As part of the options you can set the `composeOutput` property to a function that will receive the tree and the resolved dependency graph. It's then up to you to preform the concatenation and return the the tree.

### Post Build Addon Hooks
Once the tree has been concated, we then send that tree through the addon post-build hooks. This completes the the addon hook lifecycle.

# Migration Plan

## Ember CLI Developers

We need to move all of the client dependencies that Ember CLI relies on to start publishing to npm as addons and are resolve their dependencies within node_modules as oppossed to bower_components. We will also need to rename some of the modules as they are at unreachable code paths e.g. `ember/resolver` does not actually resolve to the ember namespace.

## Apps

To ease the transition away from Bower we must provide an Ember CLI addon that adds a dependency resolver for bower components.  This would use the bower.json to whitelist modules and simply pull them into the tree from bower_components. We should warn whenever a dependency is resolved this way.

Migrating to the Packager is as simple as changing the following in your Brocfile.

```
var Packager = require('ember-cli-packager');
var packager = new Packager({
  // Any exisiting EmberApp configuration plus the follow new options
  entries: ['my-app'], // Optional if you're only building your app
  strategies: ['default'], // Optional unless you want to change the concating
  composeOutput: function(tree, graph) {} // Optional
});
```

Since we are traversing a graph `app.import` for javascript dependencies should be removed.

## Addons

During this process we have slightly tweaked the structure of the app and addon folders in an addon.

### App Directory
For the app `pods` is now a directory and will get merged into the consumers namespace under `app/pods`.

### Addon Directory
Instead of only having 1 namespace per addon we have changed the directory structure to allow for N namespaces from an addon. 

The new directory structure looks like the following:

```
my-addon/
  addon/
    my-namespaced-thing/
      c.js
      d.js
    my-addon/
      a.js
      b.js
    my-addon.js
```

We will add deprecation warns on this change to allow for addons to migrate before the Packager becomes the new default.

Since the Packager introduces the ability to do selective builds "main" files that re-export are an anti-pattern.

# Other Fun Stuff

Other things that fall out of this work include:

- Use Ember like a library importing only the things you need from it. This should cause the size of your app to reduce greatly.
- Auto generate a [ServiceWorker](http://www.html5rocks.com/en/tutorials/service-worker/introduction/) for you
- Auto generate [WebWorkers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) for Ember Data serialization

These are out of the scope of this RFC, but there is now a path to them.

# Alternatives

[Webpack](http://webpack.github.io/) has become very popular in solving this similar problem. However it is unclear how WebPack could be used with Ember as Ember has both explicit and implicit dependencies. This is why we feel we need something that is aware of how Ember apps are assembled and take advantage of existing tools when it is makes sense. Since we have an opinionated way of structuring our apps, we can avoid the "wall of configuration" that other asset packagign systems are susceptible to.

# Unknowns/Risks

While NPM is more popular than Bower, it has yet to really address the peer dependency problem in which "there can only be one", otherwise known as the Highlander rule (coined by Yehuda Katz). Stef has met with the NPM folks and believes NPM 3 will help solve a slew of frontend dependency problems.