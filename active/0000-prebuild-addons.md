- Start Date: 2017-03-08
- RFC PR: (leave this empty)

# Summary

Allow Ember Addons to be "pre-built" such that they produce assets which can be included more efficiently in the build process of a consuming application.

# Motivation

When working on Ember-CLI applications, it is common to include many addons; this is one of the great benefits of the Ember ecosystem. However, in our current system, addons are built with the application they are included in. This means that for each addon you include in your application, you incur a non-trivial impact to your application's build performance.

Since most addon consumers never customize an addon's build or change its source code, it would be beneficial to provide pre-built versions of their assets to improve build performance. Doing so will benefit all applications and allow larger applications, especially those being developed with Engines (which are packaged as addons), to scale to truly ambitious levels.

# Detailed design

## Pre-builds for Addons

A new command will be introduced for use inside of addons only and used as part of the publishing process:

```bash
ember pre-build
```

This command will generate assets into a directory named `prebuilt` in the root of the addon. Within `prebuilt` will be a series of directories representing prebuilt assets for different combinations of targets and environments. Since targets can be non-primitive values, they will be hashed for the corresponding directory path.

```bash
cool-addon/
  |-prebuilt/
    |-48e380a165c417253512a904d6b5cf2b-prod/
    |-48e380a165c417253512a904d6b5cf2b-dev/
```

Within each of the prebuilt asset directories, there will be directories which contain the results of each `treeFor*` hook in the addon prior to being merged with a consuming application.

```bash
cool-addon/prebuilt/48e380a165c417253512a904d6b5cf2b-prod/
  |-addon/
  |-addon-templates/
  |-addon-test-support/
  |-app/
  |-public/
  |-styles/
  |-templates/
  |-test-support/
  |-vendor/
```

If any of the above folders would be empty, then they will be excluded from the output.

### Specifying Targets and Environments

Pre-built asset bundles will be generated for Ember-CLI's default target as well as development and production environments. Since custom targets and environments are likely to not have high reuse, they will be excluded from pre-builds to avoid penalizing all consumers during package installation.

## Consuming Pre-builds

During the build of consuming applications, if the target and environment information match one of the prebuilt asset bundles for a given addon, then those assets will be used by the build pipeline in place of the various `treeFor*` hooks that would normally execute for that addon.

If a consumer wishes to build the addons from scratch with their application, they will be able to specify an `ignorePrebuiltAddons` flag in their `ember-cli-build.js`, like so:

```js
const app = new EmberApp(defaults, {
  ignorePrebuiltAddons: true
});
```

# How We Teach This

The proposed changes here will primarily affect addon developers. Therefore, a new section will be added to the Ember-CLI ["Developing Addons and Blueprints" guides](https://ember-cli.com/extending/#developing-addons-and-blueprints) that describes how pre-building works at a high-level and how to perform it as an addon author.

Application users do not really need to be aware of these changes as they will be mostly transparent from a consumption standpoint. Though, they should notice faster build times.

Consumers that want to opt-out of pre-building will need to be educated about the `ignorePrebuiltAddons` option, which should be documented in the [API documentation for `EmberApp`](https://ember-cli.com/api/classes/EmberApp.html).

# Drawbacks

The only considerable drawbacks to these changes are increased complexity in the Ember-CLI build pipeline and slightly increased API surface area.

# Alternatives

The impact of not doing this is sub-optimal build performance, especially for larger applications or those using many addons.

## Configurability of Targets / Environments

One alternative aspect of this proposal would be to allow addon authors to customize the targets and environments for which they pre-build. While this would likely not be too difficult to implement, it risks penalties to all users due to higher installation costs with potential for minimal return.

If additional, common targets/environments become present we can roll them into the preset builds. Alternatively, we can add this feature in the future, but leaving it out for now is much easier than potentially having to unwind it.

## Pre-built Asset Format

To limit potential download and file system I/O costs, an alternative format for the pre-built assets may be considered. Using a JSON representation of the pre-built assets would allow for storage as a single, large file which can reduce the amount of overhead in the pre-builds. This approach, however, makes understandability and debuggability of the pre-built assets much less and should likely be considered after sufficient real-world data has been collected on the current suggested approach.

# Unresolved questions

- Per-target installation of addons? Is it possible to publish multiple versions of addons with pre-built assets for different targets/environments? If so, this may fix the concerns over custom pre-builds.
- For non-default targets, what happens? Are they opted out of all pre-building? Can they opt-in again if they know it is fine for their app?
