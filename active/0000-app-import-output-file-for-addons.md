- Start Date: 2016-04-03
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Allows `app.import` to specify outputFile for addons.
Not calling `app.import` for an addon would be considered to have
an outputFile of `assets/vendor.js` to maintain the existing behavior.

# Motivation

[RFC#28](https://github.com/ember-cli/rfcs/pull/28) added support to specifiy an `outputFile` this works great for static `vendor` files, however, the addon's processed JS always end up in `vendor.js`. We have the same motiviation to do this for addons as we do for other vendor assets. This is specially important for large applications where only parts of the app depend on certain addons. Splitting the output vendor files allows us to cache assets more efficiently or even lazy-load them. Note: lazy-load is out of the scope of this RFC


# Detailed design

Today the addonTree processes JS, which get contenated together to `addons.js` and later to `vendor.css` and `vendor.js` respectively. With this change, we will allow the addon consumers further control about where the processed JS output for each addon ends up. Today we don't process addon's CSS directly this way, but we already have a solution via `outputPaths`, for this proposal CSS is ignored. See alternatives for a consistent option with CSS and JS.

`outputFile:` option, specifies the target files for the given addon import. If multiple addons share an outputFile, they will be concatenated in the order they where imported, but appended to regular `app.import`s.

`outputFile:` will default to `assets/vendor.js` and `assets/vendor.css`

Today all addons defined in package.json are processed and imported. This behavior remains the same. `app.import` is used for consistency with how we import `vendor` files, but it is optional for addons and only needed if we want to override the outputFile.

While today we can't `app.import` from `node_modules`, this might be supported in the future and other consumers might already be funneling to make this work, in order to dissambiguate we import them from `addons` instead as `app.import('addons/ember-view-state', ...)`. This is also more explicit since we're importing the processed JS, instead of a specific file under `node_modules`.

## Examples


#### variation 0: the default

```js
app.import('addons/ember-lgtm', { outputFile: 'assets/vendor.js'});
```

The code above is redundant, since it is the default for all addons. This is only needed if you need to specify the order or override the `outputFile`.

#### variation 1: 1 file -> 1 outputFile

```js
app.import('addons/ember-lgtm', { outputFile: 'assets/ember-lgtm.js'});
```

* The treeForAddon for  `ember-lgtm` becomes `assets/ember-lgtml.js`
* Other addon trees
* in prod it is:
  * uglified (unless using the uglify options it is excluded)
  * fingerprinted (unless it is excluded via the asset-rev options)

#### variation 2, multiple files to same outputFile

```js
app.import('addons/ember-signature-pad.js', { outputFile: 'assets/vendor-deferred.js'});
app.import('vendor/background-update.js', { outputFile: 'assets/vendor-deferred.js'});
```

* in-order of the corresponding `app.import` invocation, using sourceMap
  concat, the files are combined into `assets/vendor-deferred.js`

#### variation n, multiple files to same outputFile

```js
app.import('addons/ember-addon-1.js', { outputFile: 'assets/alternate-vendor.js'});
app.import('addons/ember-addon-2.js', { outputFile: 'assets/alternate-vendor.js'});
app.import('addons/ember-addon-n.js', { outputFile: 'assets/alternate-vendor.js'});
```

## Trees

A typical addon have different trees with JS that are relevant for this discussion. For a `app.import('addons/ember-lgtm', { outputFile: 'assets/vendor-deferred.js'});` we will do the following:

* `app` will still be added to app.js
* `test-support` will still be part of the app's test tree.
* `addon` and `vendor` after this is implemented, it will be concatenated with other content in `vendor-deferred.js`. Note that an addon's `vendor` tree refers to `app.import`s done from the addon's `index.js`, not the `vendor` folder.


# Drawbacks

* Potential confusion about static and "addon" files. It isn't clear which ones are procesed and which ones are not.
* Potential confusion about using `addons/` as the path.
* Potential confusion about the order since addons not specified are still "imported" into vendor.js and vendor.css

# Alternatives

Each addon author could add support this, in a similar way that [ember-compsable-helpers](https://github.com/DockYard/ember-composable-helpers/blob/master/index.js#L31) supports tree-shaking, however, this will require changes on each addon and would lead to confusion and likely a non-standard way of doing it since each addon could implement this separately.

Building addons manually is possible, but it requires dropping down to broccoli, duplicating some of the work that ember-cli already does to build JS and overriding each addon's trees, doing that feels like a slightly too low level of abstraction.

## Alternative APIs

### app.addonOutput

We could introduce a new API. `app.addonOutput('addonName', 'output')` can be more explicit and lead to less confusions than using `app.import`.

### app.import(... {type: addons})

We could introduce a new API. `app.import('addonName', {type: 'addon'}, outputFile: {...}')` is more explicit, removes the confusion about the `addon/` path, but is slightly more verbose. I don't like this since it isn't really a new `type`.

### css

CSS is a bit different since CLI doesn't process it OOB, there is already some fragmentation. Some addons `app.import` this and CSS ends up in the `vendor` folder. In other cases, the addon exposes sass or less and we can use a pre-processor support and get a bit more control using `outputPaths`. For example:

```
let app = new EmberApp(defaults, {
  outputPaths: {
    app: {
      css: {
        'app': '/assets/css/app.css',
        'deferred-styles': '/assets/css/deferred-styles.css',
      }
    }
  }
});
```

In the example above, `styles/deferred-styles.scss` is simply a file with references to other CSS or SASS.

We could allow users to specify an outputFile for addons's CSS by allowing `outputFile:` to take a hash for each type `{outputFile: {css: 'assets/custom-path.css', js: 'assets/custom-path.js'}}`.


# Unresolved questions

* Should this affect import's of addons? For example, `ember-lgtm` imports `lgtm-standalone.js`. If we have `ember-lgtm` in `vendor-ember-lgtm`, but don't do anything about `lgtm-standalone` the latter will end up in vendor.js, which isn't always what we want. I don't think the consumer of an addon should be aware or manage the addon dependencies, but implicitly importing the addon's dependencies to the addon file can lead to other issues when we have other addons depending on that file. Today ember-cli will import it only once, so if we have a second addon depending on `ember-lgtm` we will end up with a single copy. However, if we implicitly import we could end up with the same dependency in more than one asset.
* Should we warn if another addon imports an addon that is on a different file? For example `ember-offline-states` depends on `ember-connection-monitor`, if we move the the latter to `vendor-deferred.js`, `ember-offline-states` can break if we only load `vendor.js`. Unfortunately, today we don't have a way to know how each asset will be used.
