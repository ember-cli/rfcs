- Start Date: (2015-07-12)
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Allow ember-cli to target another package.json file when checking its dependency list. Normally for node projects you can symlink your local node_modules folder to another location. If ember-cli's local package.json dep list is different from what's installed in that "shared" node_module directory, it will throw an error and not build. This would allow app authors to point their devDependencies (and other keys as well, in theory) to package.json located elsewhere.

# Motivation

The main idea is to allow app authors to manage their project dependencies from a different location. Normally using a symlink one can access deps installed elsewhere, but ember-cli explicitly looks at the `devDependencies` key set on its local package.json file during initialization.

# Detailed design

Imagine a complicated project structure such as this:
```
complicated-web-app/
|
|__package.json
|
|__node_modules/
|
|__ember-app-one/
|   |__package.json
|   |__node_modules/
|
|__ember-app-two/
    |__package.json
    |__node_modules/
```
During ember-cli's intialization (before any commands are actually run), the instantiates a new Project object, which maintains a series of properties determined at runtime. One of those properites is `this.pkg`, which is the local package.json file's keys and values.

This proposal would provide for a test at initialization time (`new Project()`) to look for a special key in the local package.json filed named `alternateKeys`.

### alternateKeys
In an ember-cli project's package.json, specify an array of Objects, each containing two keys: `key String` and `file String`. 

`key` is the name of the key from the other package.json file you want to set on `this.pkg`. `file` is the path to that other package.jon. The path must relative to the project root. 
```
// complicated-web-app/package.json
{ 
  "name": "complicated-web-app", 
  "version": 1.2.3, 
  "devDependencies": {
    "ember-cli": "1.13.1,
    "grunt": "0.4.5",
    // other complicated-web-app and ember deps
    },
}

// complicated-web-app/ember-app-one/package.json
{
  "name": "ember-app-one",
  "version": 0.1.1,
  "alternateKeys": [
    { "key": "devDependencies", "file": "../package.json" }
  ]
}

// imagine doing the same thing for ember-app-two
```

So this means you can put all your ember-cli deps into one package.json file at the root, tell the ember apps to look to that file for the dependencies and versions it should use, and symlink from the ember-app to the root node_modules.
```
$ cd complicated-web-app
$ npm install
$ rm -rf ember-app-one/node_modules && rm -rf ember-app-two/node_modules
$ cd ember-app-one
$ ln -s ../node_modules node_modules
$ cd ../ember-app-two
$ ln -s ../node_modules node_modules
```

# Drawbacks

- Adds some code to the `Project` initilization process
- It might be a rare use-case.

# Alternatives

- Could just leave it alone and decide that providing for ember-cli projects nested within other node projects is not a compelling enough of a use-case.


# Unresolved questions

- There might be a better way to tell ember-cli commands to use another file for reference.
