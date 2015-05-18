- Start Date: 2015-05-18
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Allow addons to clean up after their removal and to remove up other conflicting packages on install.

# Motivation

Managing addons becomes much simpler if the author doesn't need to understand dependencies that the author has had to account for.
This has been an issue with [ember-cli-mocha](https://github.com/switchfly/ember-cli-mocha) where qunit has conflicted.

# Detailed design

In an addons blueprint the author could write a `afterUninstall` hook to remove all it's dependencies:
```
  afterInstall: function() {
    var addonContext = this;

    return this.addBowerPackagesToProject([
      {
        name: 'ember-mocha',
        target: '~0.6.2'
      },
      {
        name: 'ember-cli/ember-cli-test-loader',
        target: '0.1.3'
      },
      {
        name: 'ember-cli/ember-cli-shims',
        target: '0.0.3'
      }
    ]).then(function() {
      if ('removeAddonFromProject' in addonContext) {
        return addonContext.removeAddonFromProject('ember-cli-qunit');
      }
    });
  },
  afterUninstall: function () {
     this.removeBowerPackagesFromProject([
       {name: 'ember-mocha'},
       {name: 'ember-cli/ember-cli-test-loader'},
       {name: 'ember-cli/ember-cli-shims'}
     ]);
  }
```

Adding:
- removeBowerPackagesFromProject
- removeBowerPackageFromProject
- removeAddonFromProject - calling this will call `ember d package-name`

# Drawbacks

The author needs to manage all dependendencies that may conflict, the alternatives section lists a possible neater way.

# Alternatives

Having a method that returns the dependencies that are required and ones that conflict.
When the user installs the packages ember-cli could inform the user of packages that conflict, that need updating or ones that are optional.
```
  dependencies: function () {
    return {
      required: {
        bower: [
          {
            name: 'ember-mocha',
            target: '~0.6.2'
          },
          {
            name: 'ember-cli/ember-cli-test-loader',
            target: '0.1.3'
          },
          {
            name: 'ember-cli/ember-cli-shims',
            target: '0.0.3'
          }
        ]
      },
      conflicts: {
        addons: [
          'ember-cli-qunit'
        ]
      }
    }
  }
```

# Unresolved questions

Is an interactive removal process better?

