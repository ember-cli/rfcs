- Start Date: 2015-03-01
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

When packaging, Ember applications utilize the dependency graph to include only the portions of the depedency tree actually utilized. Additionally, hooks will be exposed to enable further innovation. HTTP2 packaging, custom bundles, multiple files etc.

# Motivation

The current application assembling strategy is naive as it merely merges and concatenates input trees in a nearly undocumented manner. Although this simplicity has served us well, it does limit flexibility of the build process. We believe we can further factor the build process in a way the improves the current à la carte strategy, while providing additional flexibitilty when desired.

# Scope

- Break the build up into 3 distinct phases 
  - build
  - satisfy
  - package
- Traverse a dependency graph to gain selective builds
- Lay the infrastructure for the build-side of engines
- deprecate the hard dependency on Bower in favor NPM
- deprecate the app.import API

# Definitions

- __Builder__ - Responsible for generating an array of Broccoli trees that contain the app and all the addons.
- __Pre-Packager__ - Responsible for resolving and reducing the graph to a tree containing only reachable modules.
- __Packager__ - Responsible for taking the resolved tree and applying – default or user provided – concatenation strategies to the tree.
- __Entry__ - A entry node into the graph in which you traverse down from. This can be an app or engine.

# Detailed Design

## Builder
The Builder is a re-thinking of the existing `EmberApp` constructor function that is used in the Brocfile today.  Unlike the existing `EmberApp` constructor, the `Builder` constructor only does the following:

- Discover the addons
- Runs the addons hooks that are pre-resolved tree
- Merges an addon's app directory with the consuming app
- Transpiles ES2015-modules code to AMD

As part of the ES2015-modules transpilation we place a `dep-graph.json` at the root of each tree. This is used by the Pre-Packager to satisfy the graph. The result of this phase is an array of Broccoli trees containing all of the built assets in the app and addons.

At this point nothing is really different than what happens today besides placing the `dep-graph.json` and only running the addon hooks that need to happen at the beginning of a build.

## Pre-Packager
The Pre-Packager is a graph resolver algorithm for javascript dependencies. Another way of thinking about the Pre-Packager is a special purpose combination of [Broccoli-Filter](https://github.com/broccolijs/broccoli-filter) and [Broccoli-Merge-Trees](https://github.com/broccolijs/broccoli-merge-trees).

The Pre-Packager can resolve the following types out of the box:

- `ember-addons`
- Legacy modules from the node_modules directory (CJS)
- ES2015 modules from the node_modules directory

While the Pre-packager comes with a reasonable set of defaults, custom dependency resolvers can be written to allow resolution of modules outside of node_modules.

### Dependency Resolvers

There are 2 types of dependency resolvers in the pre-packager.

1. Resolvers that start the traversal from a static graph
2. Resolvers that must wait until the static graph has materialized

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
    ],
    "exports": [
      "default"
    ]
  },
  "example-app/initializers/ember-moment.js": {
    "imports": [
      "exports",
      "ember-moment/helpers/moment",
      "ember-moment/helpers/ago",
      "ember-moment/helpers/duration",
      "ember"
    ],
    "exports": [
      "default"
    ]
  },
  "example-app/router.js": {
    "imports": [
      "exports",
      "ember",
      "example-app/config/environment"
    ],
    "exports": [
      "default",
      "initialize"
    ]
  }
}
```

### Static Graph Resolution

App and addon resolution uses the `dep-graph.json` to start the traversal. By default, the Pre-Packager uses the App as the entry into the graph, but developers can supply N entries. More on this later, but for now you should think of an entry as a large functional area of an application (the admin section, the main app, etc).

#### Syncing
The first step in the resolution phase is to `syncForward` the entries files to what will be the output tree. Any non-javascript dependencies are synced forward as well.

#### Resolution Algorithm
The resolution algorithm is as follows:

1. Read in the `dep-graph.json` for an entry
2. Look at the first file in the graph and `syncForward` its imports
3. Take the first import, look up its graph and recurse
4. Recursion continues until the static graph is resolved

### Post-Static Graph Resolution

Legacy modules (CJS) and non-ember-addon ES2015 code coming from node_modules must come after. This is because we do not have a static graph prior to resolution and we look at these dependencies as opaque. During the static resolution we keep track of the types we need to resolve later.  This is denoted by a prefix on the import. If you're familiar with WebPack, Browserify, or RequireJS this is a similar approach.

To optionally hint at the Pre-Packager to resolve a legacy commonjs module use `npm:`, if the prefix is omitted the order a module will be choosen based on order of precedence.

```js
import numeral from 'npm:numeral';
```

When we attempt to resolve these opaque types of dependencies, we create an artificial "main" file containing the imports we saw and then just delegate to a tool that is designed to create bundles of these types.

For instance, `npm:` uses Browserify to create bundles of CJS modules. This type of resolution was first done by Edward Faulkner with [Ember Browserify](https://github.com/ef4/ember-browserify) and we feel it's the best way to bridge these types of gaps.

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
As part of the options you can set the `composeOutput` property to a function that will receive the tree and the resolved dependency graph. It's then up to you to perform the concatenation and return the tree.

### Post Build Addon Hooks
Once the tree has been concated, we then send that tree through the addon post-build hooks. This completes the the addon hook lifecycle.

# Migration Plan

## Ember CLI Developers

We need to move all of the client dependencies that Ember CLI relies on to start publishing to npm as addons and resolve their dependencies within node_modules as oppossed to bower_components. We will also need to rename some of the modules as they are at unreachable code paths e.g. `ember/resolver` does not actually resolve to the ember namespace.

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

Since we are traversing a graph, `app.import` for javascript dependencies should be removed.

## Addons

During this process we have slightly tweaked the structure of the app and addon folders in an addon.

### App Directory
For the app `pods` is now a directory and will get merged into the consumers namespace under `app/pods`.

### Addon Directory
Instead of only having 1 namespace per addon we have changed the directory structure to allow for N namespaces per addon. 

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

We will add deprecation warnings for this change to allow for addons to migrate before the Packager becomes the new default.

Since the Packager introduces the ability to do selective builds, "main" files that re-export are an anti-pattern.

# Other Fun Stuff

Other things that fall out of this work include:

- Use Ember like a library importing only the things you need from it. This should cause the size of your app to reduce greatly.
- Auto generate a [ServiceWorker](http://www.html5rocks.com/en/tutorials/service-worker/introduction/) for you
- Auto generate [WebWorkers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) for Ember Data serialization

These are out of the scope of this RFC, but there is now a path for them.

# Alternatives

[Webpack](http://webpack.github.io/) has become very popular in solving this similar problem. However it is unclear how WebPack could be used with Ember as Ember has both explicit and implicit dependencies. This is why we feel we need something that is aware of how Ember apps are assembled and take advantage of existing tools when it is makes sense. Since we have an opinionated way of structuring our apps, we can avoid the "wall of configuration" that other asset packagign systems are susceptible to.

# Unknowns/Risks

While NPM is more popular than Bower, it has yet to really address the peer dependency problem in which "there can only be one", otherwise known as the Highlander rule (coined by Yehuda Katz). Stef has met with the NPM folks and believes NPM 3 will help solve a slew of frontend dependency problems.
