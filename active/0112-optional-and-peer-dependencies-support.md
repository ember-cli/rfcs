# Peer and Optional Dependencies Support
* Start Date: 2017-10-17
* RFC PR: [ember-cli/rfcs#112](https://github.com/ember-cli/rfcs/pull/112)

# Summary

Add support to ember-cli to discover and instantiate addons that are in `optionalDependencies`  and `peerDependencies`.


# Motivation
Currently, it is not possible for an addon to leverage `peerDependencies` as a mechanism to ensure that the addon is being used in a supported environment.

For example, a recent change to `ember-cli-htmlbars-inline-precompile` required a minimum version of `ember-cli-babel`. We did properly setup a `peerDependencies` relationship however, due to `yarn` and `npm`'s lack of enforcement of peer dependencies users of `ember-cli-htmlbars-inline-precompile`  were commonly operating in a misconfigured setup without even knowing. 

Another example is the difference between how `npm` and `yarn` treat `optionalDependencies`. Under `npm` `optionalDependencies` are _moved_ to `dependencies` during installation, however in `yarn` environments the `optionalDependencies` are not added to `dependencies` during installation. The result of this is that when an addon with an optional dependency on another addon is installed with `npm` the optional dependency will be instantiated just like any other dependency of the addon, however under `yarn` (even if the optional dependency was installed into `node_modules`) the nested addon would never be instantiated.

# Detailed design
Make `ember-cli`'s addon discovery and instantiation process aware of `optionalDependencies` and `peerDependencies`.  Specifically:

* When `ember-cli` identifies an addon with `peerDependencies` it will:
	* Validate that the peer dependency is satisfied (e.g. a `resolve.sync` from the addon's directory is able to find the peer dependency with a valid version). If not, an error is thrown
	* Instantiate the peer dependency, and include it into the `addons` array
* When `ember-cli` identifies an addon with `optionalDependencies` it will:
	* Detect if the optional dependency is available (via `resolve.sync` from the addon's root)
	* Detect if the package was already added to `dependencies` (by way of `npm install`), if it is do nothing (as the optional dependency will be handled automatically by the existing system that operates on `dependencies`)
	* If the optional dependency is available and is not already listed in `dependencies`, instantiate it and add  it to the `addons` array

# How We Teach This
This change brings things much closer to what folks would expect by default, and therefore should not add additional confusion to folks. 

Nevertheless, the addon initialization system is somewhat under documented, and not well understood. This change would need to ensure to update the API docs (in some cause writing them from scratch) for both the `AddonDiscovery` model and `AddonFactory` models. 

# Drawbacks
Unsure.  

# Alternatives
Continue to ignore `optionalDependencies` and `peerDependencies` as we do today. This leaves `optionalDependencies` in a _really_ weird state (essentially they work for `npm` users but not `yarn` users), and makes using `peerDependencies` next to impossible.

# Unresolved questions
