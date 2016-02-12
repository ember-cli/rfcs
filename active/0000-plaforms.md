- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

The ember-cli platform flag treats platforms as build targets, and gives you a
workflow for producing builds customized to a platform, as well as streamlined tooling,
enhanced resuability, and better separation of concerns for applications that
target more than one platform.  Platforms are roughly equivalent to web views,
or a shell which provides a web view, such as a browser.

# Motivation

There is a growing trend of shipping Ember applications inside of WebView shells to
provide installable cross-platform applications and to gain access to native features.

Currently, each platform has a homegrown solution for managing build/serve/test/deploy
processes.  We could reduce the surface area of commands these platforms expose and
ease the building of addons specific to a platform by enabling ember-cli to understand
the existence of platforms and how various assets relate to them.

# Detailed design

## Usage

These are the docs for using `ember-cli-platforms` with an installed platform plugin.
For the docs on creating a platform plugin, [skip ahead](./README.md#creating-plugins).

**Create a platform.**
```cli
ember platform <type> [-t "<name>"]
```

Generates a new platform of `<type>` with an optional `<name>` parameter used when more than
one instance of a platform is desired.

With Cordova this might look like the following.

Default setup (one project)
```cli
ember platform cordova
```

Advanced setup (multiple projects)
```cli
ember platform cordova -t "android"
ember platform cordova -t "ios"
```

For all commands, `platform` can be abbreviated as `p`, and
commands that mirror `ember-cli` commands preserve the `ember-cli`
shorthands for those methods as well.

**Serve a specific platform.**
```cli
ember s -p "<type>[-<name>]"
```

This is shorthand for:
```cli
ember serve --platform="<type>" [--target="<name>"] --environment="development"
```

**Build a specific platform.**
```cli
ember b -p "<type>[-<name>]"
```

**Test a specific platform.**
```cli
ember t -p "<type>[-<name>]"
```

**Deploy a specific platform.**
```cli
ember d <deployTarget> -p "<type>[-<name>]" 
```

This is short hand for:
```cli
ember deploy --deployTarget="<deployTarget>" --platform="<type>" [--target="<name>"] --environment="development"
```


### Platform Specific Code

Platforms would leverage in-repo-addon semantics and Broccoli plugins to enable
sprinkling of platform specific code into an app as needed with "platform addons".

A platform addon is generated automatically for each platform created with `ember platform`.

```
<project>
  /lib
    /<type>[-<name>]
```

These "addons" function differently from normal addons, and would be prefixed with `-platform-`.
Instead of code in the app overriding or importing code from the addon, the addon would 
take precedence.  This let's you selectively replace or add entire modules as needed.

You can import the corresponding `app` version of a module into the platform addon.

```js
import Foo from '<project-name>/routes/foo';

// our platform version of routes/foo
export default Foo.extend({
  hello: 'World!'
});
```

### Platform Specific Config

A platform config file is generated automatically for each platform you create with `ember platform`.
```
<project>
  /config
    /platforms
      /<type>[-<name>].js
```

This configuration will be merged with your primary configuration.

## Plugins

Users should be able to create platform plugins as addons which implement 
any number of specified hooks. These plugins should carry in the same tradition
in form, function and hook names as `ember-cli-deploy`.

## Creating Plugins

Platforms have several forms of plugins:

1. `process hooks` which add behaviors to one or more platforms.
2. `platform plugins` which implement support and generators for a specific platform.
3. `deploy plugins` which implement support for deployment of a platform.

Platform plugins (generally) are also process hooks.

## All plugins

Platform plugins are ember-cli addons with a few platform specific traits:

1. they contain a package.json keyword to identify them as plugins
2. they are named ember-cli-platform-*
3. they return an object that implements one or more of the ember-cli-platforms pipeline hooks


### Creating a Process Hook

#### Create an addon

```cli
ember addon ember-cli-platform-<name>
```

#### Mark the addon as a plugin

Identify your addon as a plugin by updating the package.json like so:

```js
// package.json

"keywords": [
  "ember-addon",

  // if present, indicates this addon contains process hooks for ember-cli-platforms
  "ember-cli-platform-plugin",
  
  // if present, indicates this addon contains a platform implementation for ember-cli-platforms
  "ember-cli-platform",
  
  // if present, indicates this addon implements deployment hooks
  "ember-cli-platform-deploy"
]
```


#### Implement one or more pipeline hooks

In order for a plugin to be useful it must implement one or more of the ember-cli-platforms
pipeline hooks.

To do this you must implement a function called `createPlatformPlugin` in your `index.js` file.
This function must return an object that contains:

1. A name property which is what your plugin will be referred to.
2. One or more implemented pipeline hooks

**Example:**

```js
module.exports = {
  name: 'ember-cli-platform-live-reload',

  createPlatformPlugin: function(options) {
    return {
      name: "live-reload",

      didBuild: function(context) {
        //do something amazing here once the project has been built
      }
    };
  }

};
```

#### Process Hooks

- configure
- setup
- willBuild
- build
- didBuild
- teardown



### Creating a Deployment Plugin

This is built overtop of [ember-cli-deploy](http://ember-cli.com/ember-cli-deploy/docs/v0.5.x/pipeline-hooks/),
but integrates with your platform flow.

To create a deploy plugin, follow the same steps as for `process hooks`, making sure to
add the keyword `ember-cli-platform-deploy` to `package.json`. The following is a full list
of hooks which can be returned by `createPlatformPlugin` for use with deployment.

#### All Hooks (including deployment)

- configure
- setup
- willDeploy
- willBuild, build, didBuild,
- willPrepare, prepare, didPrepare,
- willUpload, upload, didUpload,
- willActivate, activate, didActivate, (only if --activate flag is passed)
- didDeploy,
- teardown



### Creating a Platform Plugin

Platform plugins have several concerns they need to manage.  They should

- implement the build hooks of the pipeline
- provide a starter kit for the platform
- provide a CLI proxy for SDK commands
- provide a blueprint for project installation
- run any necessary SDK installation commands
- handle SDK specific file inclusion

**Command Runners**

- ember-cli-platform supplied a commandRunner you can use to run CLI commands directly from
  the pipeline if necessary.

**Command Proxies**

- If your ember-cli-platform-plugin specifies a `commandName`, it will be used to generate
 a command proxy for that platform.  For instance:
 
```cli
{
  commandName: 'cordova'
}
```

If a user generates a cordova platform with the following command

```cli
ember platform cordova -t "ios"
```

Then an `npm` command script will be generated and installed that proxies use of
`cordova-ios` into the cordova project directory ios, allowing you to easily use
the cordova-cli.

**Minimum Settings**
- `outputDestination`
- `addon-name`


# Drawbacks

Security risks.  Pipeline Concept.  Difficult to resolve Test/Serve situations.

# Alternatives

Leave this to the addon ecosystem, potentially with each platform implementing bespoke
solutions.

# Unresolved questions

The following concerns will need to be fleshed out to properly provide
an abstract process for each platform to be able to utilize.

- where do we put the (static) config for the commandName?
- ember serve doesn't resolve promise when serving, nw.js had to hook it
- testing: run tests within the deployment target
  - view based tests
  - special test runner
  - special qunit adapter for testem
  - special test command
- testing: run tests for each platform on travis/circle/etc.
- a way to configure `environment.js` if needed (beyond the platform config file, probably just a merge)
- live reload
  - likely: platform plugins would implement hooks to allow you to live reload when working with simulators
or attached devices.

> N.B. here's how ember-cli-cordova was approaching this
> - ember-cli-remote-livereload
> - auto add `<allow-navigation href="http://10.0.2.2:4200/*" />`
> - cordova.liveReload.enabled = true
> - cordova.liveReload.platform = 'android'
> - cordova.emberUrl
> - PR https://github.com/poetic/ember-cli-cordova/pull/56

- Support for Ember Inspector for devices?
  - Maybe. There's a project specifically for this [ember-cli-remote-inspector](https://github.com/joostdevries/ember-cli-remote-inspector)


## Wish List

- develop multiple projects with live-reload at once
