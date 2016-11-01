- Start Date: 2016-11-10
- RFC PR: (leave this empty)
- Ember CLI Issue: (leave this empty)

# Summary

This RFC will primarily allow fastboot assets to be built using `ember build`. Fastboot is a different environment in which your ember apps run and require your app behavior to change a bit in this environment that running your app in the browser. This requires building fastboot assets that compliment the current browser assets.

This RFC describes the details of how fastboot will be built inside `ember-cli`. Since we do not know the generic usecases, we would like to build this `ember-cli` as the first pass. Once we have understood the usecases, we can make it generic per this [RFC](https://github.com/ember-cli/rfcs/pull/75). This RFC aims to expose the minimal public API and work to build fastboot in `ember-cli`.

# Motivation
[Fastboot](https://ember-fastboot.com) allows ember apps to be rendered on the server side. It renders the initial content of your app allowing your app to be used in SEO usecases. On a very high level, fastboot loads your assets in a Node environment and creates the Ember App once the assets are loaded. It does not do any automatic routing but creates Ember application instance whenever there is a server side rendering request and runs your route to render the initial render in Node.

In order for your app/addon to be able to run in a Node environment, it needs to be compatible in such environment. Compatibility includes not doing DOM/window access in Node, making sure you are able to make API requests in Node etc. Therefore an app/addon may need to provide fastboot specific initializers, instance-initializers, services etc that are only applicable in the Node environment and not in the browser. Including them in the browser has performance implication since you will send down more bytes to the client.

Currently fastboot builds the app twice and generates a different set of assets for fastboot and browser environments via ember fastboot command. Most of the app between the two sets of assets is same except where we want fastboot to override the browser behavior or need fastboot specific handling. Currently ember fastboot filters these and generates the two sets of assets. Building different sets of assets (app and vendor files) for different environments cannot guarantee the app works correctly in both environments always. Moreover, ember-fastboot --serve-assets command also spins up a node server and serves your assets. This is very similar to what ember serve does today. However, we end up forking and creating two parallel worlds with problems such as serving stale assets, creating race conditions during serving and building etc. We want to be able converge the two worlds such that fastboot should be able to specify the last middleware that will be used to serve the assets in fastboot. This helps us not create fastboot commands and keeps the eco-system unified.

We want to build these fastboot specific code as an additive asset to the browser that loads in the Node VM in addtion to the the other assets. If it overrides any browser behavior, having the same module id will override the definition in the loader. In order to build fastboot assets in a performant way, we would like to integrate building fastboot into ember-cli. This issue specifically tracks integrating building fastboot in ember-cli. When we need to build more such artifacts, we can extend this to be a generic target solution.

# Detailed design

## Directory Structure for fastboot app/addons
In order to build the fastboot assets which are an addition to the app and vendor files, we want to define a separate directory structure. This will allow fastboot specific app code to live in its own namespace and does not touch the browser specific app file.

Currently today no fastboot specific code lives in the addon namespace. Hence we do not need `fastboot/addon` namespace as part of this RFC in `ember-cli`.

For example an addon containing fastboot specific functionality will have a directory structure like:
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

This allows us to keep fastboot specific functionality in its own namespace and allows us to build fastboot artifact by walking only the subset of the tree rather than walking the whole app and doing filtering (as done right now).

After an ember build, the generated assets will be:
1. vendor.js
2. app.js
3. app-fastboot.js

In the Node VM, we will first load the assets and eval them in the above order. Therefore, if you wanted to override a functionality in fastboot, you can keep the name of the file same in fastboot namespace and will override it when app-fastboot.js gets loaded and evaluated. This also guarantees that we do not regress existing ember builds that do not contain the fastboot namespace.

## Public API
With building the additional assets, this RFC would also like to expose some public APIs so that addons that are doing fastboot specific things can leverage that to build fastboot assets. The additional public API will be as follows:
1. `preprocessTree(type, tree)`: Currently the type includes only for browser specific assets. We want to invoke this hook for fastboot asset as well. Therefore, when fastboot assets are built, this hook will be invoked with type as `fastboot`. Addons can use this hook to munge the fastboot/app tree before it gets processed.
2. `postprocessTree(type, tree)`: This currently is also only invoked for browser specific assets. We want to invoke the same hook with `fastboot` as the type when building fastboot asset.
3. `treeForFastbootApp(tree)`: This is a new hook that this RFC proposes to add which can be used by addons to munge the tree for `fastboot/app` before it is processed.

# How We Teach This

This is an advanced feature and any app that defines `fastboot/app` and runs `ember build` will end up generating an additional asset as `app-fastboot.js`. We would need to update the `ember-cli` documentation to explain users how this additional asset is generated and how it will be of limited use.

Moreover, `ember-cli-fastboot` will do the work of moving the existing fastboot specific funtionality in various addons into `fastboot/app`. As part of building fastboot assets faster, we will deprecate the `ember fastboot` command.

# Drawbacks

The only drawback this has is that it builds fastboot specific code in `ember-cli`. This is only being done since we do not want to come up with a generic solution until we fully know the usecases to make this generic that any additional asset can be built.

# Alternatives

An alternative [RFC](https://github.com/ember-cli/rfcs/pull/75) which is a more generic solution and long term goal in `ember-cli`.

# Unresolved questions

N/A
