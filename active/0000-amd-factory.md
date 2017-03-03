- Start Date: (fill me in with today's date, 2016-06-14)
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

This RFC proposes extending the `app.import` API to simplify importing external libraries that do not include a `define` or `module.exports` loader.

# Motivation

A large number of jquery plugins and other external libraries do not contain a  `define` or `module.exports` loader and add their exports
directly to the global scope (`window`). In order to use such a plugin in Ember, one has to write an addon. This
can be a threshold for people. We should aim to make importing external libraries as easy as possible.

With the [anonymous AMD loader](https://github.com/ember-cli/ember-cli/pull/5976) and the [CommonJS loader support](https://github.com/ember-cli/ember-cli/pull/6812) big
steps have already been taken. This proposal builds on those features.

# Detailed design

Libraries that are added to the global scope, require another wrapper to conform to the AMD module loader syntax. This
wrapper should be fixes missing import and export statements.

## The wrapper
A wrapper might look as follows.

```js
// the contents of these three variables dependent on the arguments passed to app.import, see below
let dependencies = '["' + ['jquery'].join('","') + '"], ';
let variables = '["' + ['jQuery'].join('","') + '"], ';
let returnVariable = 'return window.externalLibrary;';

// wrapper
content = [
  '(function(factory) {\n',
  'define(' + dependencies + 'factory);',
  '})(function (' + variables + ') {',
  content,
  returnVariable,
  '});',
].join('');
```

## Missing imports
Example for a library that leans on a global `jQuery` being defined.

```js
(function ($) {
  $.fn.externalLibrary = function () {};
})(jQuery);
```

It is not possible to import this library correctly. By extending the `app.import` method and the wrapper above, this
should become possible without creating an addon.

```js
app.import('/path/to/module.js', {
  using: [
    {
      transformation: 'amd',
      as: 'some-dep',
      'import': {'jquery': 'jQuery'},
    }
  ]
});
```

## Missing exports
Example for a library that only registers itself to the global scope

```js
(function (global) {
  global.externalLibrary = function () {};
})(window);
```

It is not possible to import this library without creating an addon. By extending the `app.import` method and the wrapper
above, this should become possible without creating an addon.

```js
app.import('/path/to/module.js', {
  using: [
    {
      transformation: 'amd',
      as: 'some-dep',
      'export': 'window.externalLibrary',
    }
  ]
});
```


# Learning

There should be extensive documentation how this works.

# Drawbacks

People have less reason to create an addon.