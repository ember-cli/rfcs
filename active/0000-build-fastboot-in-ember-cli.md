- Start Date: 2016-11-10
- RFC PR: (leave this empty)
- Ember CLI Issue: (leave this empty)

# Summary

This RFC will primarily allow FastBoot assets to be built using `ember build`. FastBoot is a different environment in which your ember apps run and require your app behavior to change a bit in this environment that running your app in the browser. This requires building FastBoot assets that compliment the current browser assets.

This RFC describes the details of how FastBoot will be built inside `ember-cli`. Since we do not know the generic usecases, we would like to build this into `ember-cli` as the first pass. Once we have understood the usecases, we can make it generic per this [RFC](https://github.com/ember-cli/rfcs/pull/75). This RFC aims to expose the minimal public API and work to build FastBoot in `ember-cli`.

# Motivation
[FastBoot](https://ember-FastBoot.com) allows ember apps to be rendered on the server side. It renders the initial content of your app allowing your app to be used in SEO usecases. On a very high level, FastBoot loads your assets in a Node environment and creates the Ember App once the assets are loaded. It does not do any automatic routing but creates `Ember.ApplicationInstance` whenever there is a server side rendering request and runs your route to render the initial render in Node.

In order for your app/addon to be able to run in a Node environment, it needs to be compatible in such an environment. Compatibility includes not doing DOM/window access in Node, making sure you are able to make API requests in Node etc. Therefore an app/addon may need to provide FastBoot specific initializers, instance-initializers, services etc that are only applicable in the Node environment and not in the browser. Including them in the browser has performance implication since you could be sending down more bytes than required to the client at runtime.

Currently FastBoot builds the app twice and generates a different set of assets for FastBoot and browser environments via `ember fastboot` command. Most of the app between the two sets of assets is same except where we want FastBoot to override the browser behavior or need FastBoot specific handling. Currently `ember fastboot` filters these and generates the two sets of assets (each for a different environment). Due to filtering and creating two different sets of assets we end up running the build on the app twice. Building different sets of assets (app and vendor files) for different environments cannot guarantee the app works correctly in both environments always. This RFC helps us not create FastBoot commands (which patches the private API of `ember-cli`, does a double build to generate two sets of assets and does a heavy filtering in the app tree causing large build times) for build and keeps the eco-system unified.

We want to build this FastBoot specific code as an additive asset to the browser that loads in the Node VM in addition to the the other assets. If it overrides any browser behavior, having the same module id will override the definition in the loader. In order to build FastBoot assets in a performant way, we would like to integrate building FastBoot into ember-cli. This issue specifically tracks integrating building FastBoot in ember-cli. When we need to build more such artifacts, we can extend this to be a generic target solution.

# Detailed design

## Directory Structure for FastBoot app/addons
In order to build the FastBoot assets which are an addition to the app and vendor files, we want to define a separate directory structure. This will allow FastBoot specific app code to live in its own namespace and does not touch the browser specific app file.

*Note*: Currently today no FastBoot specific code lives in the addon namespace. Hence we do not need `FastBoot/addon` namespace and building of that namespace as part of this RFC.

For example an addon containing FastBoot specific functionality will have a directory structure like:
```
my-addon/
|-- addon/
│   |-- utils/
│   │   |-- foo.js
│-- app/
│   │-- instance-initializers/
│   │   |-- error-handler.js
│-- fastboot/
│   │-- app/
│   │   |-- instance-initializers/
|   |   |   |-- error-handler.js
|   |   |   |-- ajax.js
```
Note: The same applies for an app as well.

This allows us to keep FastBoot specific functionality in its own namespace and allows us to build FastBoot artifact by walking only the subset of the tree rather than walking the whole app and doing filtering (as done right now).

After an ember build, the generated assets will be:
1. vendor.js
2. app.js
3. app-fastboot.js

In the Node VM, we will first load the assets and eval them in the above order. Therefore, if you wanted to override a functionality in FastBoot, you can keep the name of the file same in FastBoot namespace and will override it when app-fastboot.js gets loaded and evaluated. This also guarantees that we do not regress existing ember builds that do not contain the FastBoot namespace.

## Public API
With building the additional assets, this RFC would also like to expose some public APIs so that addons that are doing FastBoot specific things can leverage that to build FastBoot assets. The additional public API will be as follows:
1. `preprocessTree(type, tree)`: Currently the type includes only for browser specific assets. We want to invoke this hook for FastBoot asset as well. Therefore, when FastBoot assets are built, this hook will be invoked with type as `app-fastboot`. Addons can use this hook to munge the FastBoot/app tree before it gets processed.
2. `postprocessTree(type, tree)`: This currently is also only invoked for browser specific assets. We want to invoke the same hook with `app-fastboot` as the type when building FastBoot asset.
3. `treeForFastBootApp(tree)`: This is a new hook that this RFC proposes to add which can be used by addons to munge the tree for `fastboot/app` before it is processed.

# How We Teach This

This is an advanced feature and any app that defines `fastboot/app` and runs `ember build` will end up generating an additional asset as `app-fastboot.js`. We would need to update the `ember-cli` documentation to explain users how this additional asset is generated and how it will be of limited use.

Moreover, `ember-cli-fastboot` will do the work of moving the existing FastBoot specific funtionality in various addons into `fastBoot/app`. As part of building FastBoot assets faster, we will deprecate the `ember fastBoot` command.

We will also be upgrading [FastBoot](https://ember-fastboot.com) website of how this is works with `ember-cli`.

# Drawbacks

Following are the drawbacks of this approach:
- [ ] It builds FastBoot specific code in `ember-cli`. This is only being done since we do not want to come up with a generic solution until we fully know the usecases to make this generic that any additional asset can be built.
- [ ] This approach of loading additive assets after the main asset heavily depends on the loader.js behavior of "last one" wins always. If this behavior changes in future, this approach will break.

# Alternatives

An alternative [RFC](https://github.com/ember-cli/rfcs/pull/75) which is a more generic solution and long term goal in `ember-cli`.

# Unresolved questions

- [ ] How will this work in long term with module unification?
