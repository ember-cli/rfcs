- Start Date: 2016-10-31
- RFC PR: (leave this empty)
- Ember CLI Issue: (leave this empty)

# Summary

This RFC gives Ember developers to build other platform assets in addition to building the standard browser assets. It is an optional feature and should not affect the build times or the CLI usability for apps that only intend to build browser assets. To limit the scope of this RFC, this RFC only focusses on creating and building targets.

# Motivation

Currently ember-cli only supports building ember apps that are targeted for the browser environments only. At times, you may want to build additional assets that compliment to the existing browser assets. These additional assets may have the same structure of apps and addons but are meant to override the browser behavior in a different environment. Two examples of this are [ember-electron](https://github.com/felixrieseberg/ember-electron) and [ember-cli-fastboot](https://github.com/ember-fastboot/ember-cli-fastboot). This RFC primarily only addresses the usecases of fastboot but is generic enough to extend to other platforms.

[Fastboot](http://ember-fastboot.com/) allows ember apps to be rendered on the server side. It renders the initial content of your app allowing your app to be used in SEO usecases. On a very high level, fastboot loads your assets in a Node environment and creates the Ember App once the assets are loaded. It does not do any automatic routing but creates Ember application instance whenever there is a server side rendering request and runs your route to render the initial render in Node.

In order for your app/addon to be able to run in a Node environment, it needs to be compatible in such environment. Compatibility includes not doing DOM/window access in Node, making sure you are able to make API requests in Node etc. Therefore an app/addon may need to provide fastboot specific initializers, instance-initializers, services etc that are only applicable in the Node environment and not in the browser. Including them in the browser has performance implication since you will send down more bytes to the client.

Currently fastboot builds the app twice and generates a different set of assets for fastboot and browser environments via ember fastboot command. Most of the app between the two sets of assets is same except where we want fastboot to override the browser behavior or need fastboot specific handling. Currently ember fastboot filters these and generates the two sets of assets. Building different sets of assets (app and vendor files) for different environments cannot guarantee the app works correctly in both environments always. Moreover, ember-fastboot --serve-assets command also spins up a node server and serves your assets. This is very similar to what ember serve does today. However, we end up forking and creating two parallel worlds with problems such as serving stale assets, creating race conditions during serving and building etc. We want to be able converge the two worlds such that fastboot should be able to specify the last middleware that will be used to serve the assets in fastboot. This helps us not create fastboot commands and keeps the eco-system unified.

We want to build these fastboot specific code as an additive asset to the browser that loads in the Node VM in addition to the the other assets. If it overrides any browser behavior, having the same module id will override the definition in the loader. For platforms, that want to sprinkle the overrides back in built app.js, they can always chose to merge the two files.

# Detailed design

The detailed design is separated in various sections so that it is easier for a reader to understand.

## Directory structure for additional targets

By default and in order to maintain backward compatibility we will assume in any addon/app the following directory structure represents the `browser` target. The `browser` target will be the default mode that any app gets when creating an app/addon. The directory structure for the `browser` target in addon will be:
```
my-addon/
|-- addon/
│   |-- utils/
│   │   |-- foo.js
│-- app/
│   │-- instance-initializers/
│   │   |-- error-handler.js
```
`app` and `addon` directories in an addon or `app` directory in an app represent the browser target. In future with module normalization, we should namespace this under `browser` target to be clearer.

In order to build the additional target (for example `fastboot`) which are an addition to the app and vendor files, we want to define a separate directory structure. This will allow target specific app/addon code to live in its own namespace and does not touch the app and vendor files.

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
|   |-- addon/
|   |   |-- utils/
|   |   |   |-- foo.js
|   |   |   |-- fastboot-utils.js
```
Note: The same applies within an app as well, just that it wouldn't have fastboot/addon folder.

This allows us to keep target specific functionality in its own namespace and allows us to build target artifact by walking only the subset of the tree rather than walking the whole app and doing filtering (as done right now for fastboot builds).

After an ember build for that target (see details below), the generated assets will be (under dist/assets):

- vendor.js
- app.js
- vendor-fastboot.js
- app-fastboot.js

For fastboot, in the Node VM, we will first load the assets and eval them in the above order. Therefore, if you wanted to override a functionality in fastboot, you can keep the name of the file same in fastboot namespace and the loader.js will override it when app-fastboot.js gets loaded and evaluated. This also guarantees that we do not regress existing ember builds that do not contain the fastboot namespace. For any other targets, they can choose to load the assets in a particular order or chose to merge the assets into one such that you only load one asset in the browser.


## API to register an additional target

In order for an app or addon to define an additional target to the ember-cli eco-system, it needs to register one before the build starts. This will allow the ember-cli to be aware of the a new target that may need to be built using the optional CLI options. In addition to defining a new target, we also want to allow the addon/app the defines the target to define a dependency that this target may have on other targets. For the purpose of understanding this RFC, we will call the current default target that `ember-cli` builds as the `browser` target.

If an addon wants to define a new target, it will specify the new target in its package.json (under `ember-addon` config). The definition would look as follows in the package.json:
```
"ember-addon": {
  "configPath": "tests/dummy/config",
  "additionalTargets": {
    "fastboot": {
      "dependsOn": ["browser"]
    }
  }
}
```
The above config means that this ember-addon wants to define an additional target named "fastboot" which depends on the "browser" target. The depends on means that whenever this target is built, it needs to make sure the dependent target is also built so that the assets can be served correctly in the target environment.

If an app wants to define a new target, it will specify the new target in `ember-cli-build.js` when it creates the ember app:
```
var app = new EmberApp(defaults, {
  minifyJS: {
    enabled: false
  },
  minifyCSS: {
    enabled: false
  },
  additionalTargets: {
    "fastboot": {
      "dependsOn": ["browser"]
    }
  }
});

//...
return app.toTree();
```

## Additional options to existing CLI commands
The existing ember-cli commands only assume there is one target that it needs to build/generate files for. With the concept of targets, we now want to now allow the CLI commands to allow targets to be generated and created. The following will be the workflow for a ember user intending to create and use target.

1. A user can generate a target using the following command: `ember generate target <target name>`
This will end up creating a `fastboot` directory with app folder inside if it is done in an app or app and addon directories if done in an addon. Note: The target name should be the same as the one registered by the app/addon.

2. A user can generate a route in a target environment: `ember generate route <route name> --target <target name>`
This generates a route in the targeted environment.

3. Building a target: `ember build --target <target name>`
This will build the target artifacts. If the target depends on other targets, those will also be built as part of this command. The generated assets will all be under `dist/assets` with the following file naming:
```
dist/
|-- assets/
│   |-- vendor.js
│   |-- app.js
│   |-- vendor-<targetname>.js
│   |-- app-<targetname>.js
```

Note:
1. The command `ember serve` will remain as it is today since it serves the assets based on what is requested. If the targeted assets need to be served, the app/addon should update the index.html or their environment in which this is loaded to be served. The targeted app and addon directories should be watched so that a change in them triggers a rebuild.
2. If a target is not defined and a user uses the above command, we will throw a self explanatory error message.
3. `vendor-<targetname>.js` file will only contain the AMD modules for building addons and not the ember-cli/ember.js trees. These are already present in the vendor file and we expect users of this feature to be using the additional targets as a compliment to the existing ones.

## Public hooks
`ember-cli` as part of the build pipeline exposes public API for addons to change the different trees and processing per their needs. Since the targeted builds are doing an additional builds as a compliment to the existing browser build, we need to expose similar APIs for the target builds as well. Following are the public apis we need to expose:

- `preProcessTree(type, tree)`: The existing type for browser build is represented by `javascript`, `styles` etc. We need to expose additional types when targetted builds are run. Therefore, for a target build the type will be invoked as `javascript-<targetname>`, `styles-<targetname>` etc.
- `postprocessTree(type, tree)`: The existing type for browser build is represented by `javascript`, `styles` etc. We need to expose additional types when targetted builds are run. Therefore, for a target build the type will be invoked as `javascript-<targetname>`, `styles-<targetname>` etc.
- `lintTree(treeType, tree)`: We also want to make sure we lint the new target trees as well. Therefore we need to generate additional tree types based on the given target. The additional treeType will look as `app-<targetName>`, `templates-<targetname>` and `addon-<targetname>`.
- `treeFor(name)`: We want to expose the similar `treeFor` hooks that are available for app, addon, templates and styles only. Other names will not apply for target builds. Some examples include vendor, public etc. Therefore additional names and their corresponding public function will be available. For example a target named as `fastboot` will invoke the additional names with `treeFor`: `app-fastboot`, `addon-fastboot`, `styles-fastboot`, `templates-fastboot`, `addon-templates-fastboot` and `addon-fastboot`. The corresponding treeFor public trees will be: `treeForFastbootApp`, `treeForFastbootAddon` etc.


## app.import
`app.import` allows libraries present in vendor directory or other places to be imported in the vendor file. A target may want to import a platform specific library in its vendor file. In order to allow that we need to expand the API of `app.import()` to take into account importing into target environments.

Currently `app.import` takes an optional object that allows to provide additional options (for example, whether to prepend). The `options` object will now take a `targets` key which will be an array of target names where this asset needs to be imported. It will default to `browser` target if not provided.

_Note_: This is optional item for this RFC.

## config environments
By default ember exposes an environment file which contains configuration in different environments (development, test and production). If a target wants to override a configuration in its environment, it should be allowed to do so. Therefore, this RFC proposes an additional directory structure
```
config/
  |-- environment.js
  |-- <targetname>
    |-- environment.js
```

The `targetname` directory will only exists if the target environment wants to override. For the target build if the target environment file is present we will generate a new config and append it to `vendor-<targetname>.js` file, otherwise it will default to the browser environment config present in `vendor.js`.

_Note_: This is optional item for this RFC.

# How We Teach This

This is an optional feature that only certain addon/apps want to have. This is a backward compatible change to the existing ember eco system. I would like to propose introduce the concept of `targets` since it makes it clear there are additional target that an addon/app may introduce and want to use. Since this allows to create a parallel additional build it keeps the API and understanding to the user same as before. In order to teach users how to use targets, we need to update the API docs with a section for this and the best practices of when to use this.

# Drawbacks

Even though this proposal is very transparent to the existing ember-cli semantics, it does have two main drawbacks:
1. This generates additive assets which can be only used with the browser builds only. In order for the additive assets to override the existing behavior of the browser targets, we assume the loader overrides the modules with the same name based on the ordering. If the behavior of the loader changes in future, this will break the concept of creating additive asset.
2. If you are loading the additive assets in script tags (not valid for fastboot usecase) and for some reason the network request for fetching the additive asset fails, your app will be in an inconsistent state for that target environment. In order to play safe, the addon could merge the generated app.js and app-<targetname>.js files. This is more of a design design decision of the target environment that wants to use the additional generated assets.

# Alternatives

An alternative to the above approach is we merge the app and app-target trees with overwrite set to true so that the overriden functionality works. I have not evaluated the performance impact of this which I will do. However, it also has a disadvantage of forking the targeting environment and creating two different set of assets which may cause race conditions and will need us to introduce target option when serving files. It also creates a more complex UX for the ember-cli user and complicates debugging in target environments.

# Unresolved questions

1. Should we have CLI commands to generate targets? Ideally I don't see target environments overriding the main files such as app.js, router.js so it may be confusing to use the generate command.
2. With module unification, we should explicitly create a namespace for the target type (including the default browser type)?
3. How would tests be added for generated routes, templates etc for target environments?

# Thanks

Thanks to @stefanpenner, @nathanhammond and @kellyselden for guiding me with my first RFC.
