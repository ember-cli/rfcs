- Start Date: (fill me in with today's date, 2017-02-27)
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

This RFC proposes implementing a dynamic import syntax transformation in Ember CLI.

# Motivation

Recently, the proposal for adding a "function-like" import() module loading syntactic form to JavaScript has
reached stage 3 of the TC39 process. Other asset pipeline tools like
Webpack [already support the `import()`](https://github.com/airbnb/babel-plugin-dynamic-import-webpack) syntax.

For Ember users it can be quite useful to dynamically import modules. This is well explained in the proposal.

> This could be because of factors only known at runtime (such as the user's language), for performance reasons
(not loading code until it is likely to be used), or for robustness reasons (surviving failure to load a non-critical
module). Such dynamic code-loading has a long history, especially on the web, but also in Node.js (to delay startup
costs). The existing import syntax does not support such use cases.
>
>Truly dynamic code loading also enables advanced scenarios, such as racing multiple modules against each other and choosing the first to successfully load.

# Implementation

Once Ember CLI [supports Babel 6](https://github.com/ember-cli/ember-cli/issues/5015), it should be quite easy to
implement this RFC By using the [Babel plugin to transpile `import()` to `require`](https://github.com/genkgo/babel-plugin-dynamic-import-amd).

# Examples

## Different import based on data from the server.

This example highlights that which module to import is dependent on some data that is coming from the server.

```js
import Ember from 'ember';

export default Ember.Service.extend({
  someMethodOnThisService() {
    return Ember.RSVP.Promise((resolve, reject) => {
      fetch('/resource').then((data) => {
        if (data.variable === 1) {
          import('app/services/x').then(service => resolve(service.default), reject);
        } else if (data.variable === 2) {
          import('app/services/y').then(service => resolve(service.default), reject);
        } else {
          import(`app/services/service-${data.variable}`).then(service => resolve(service.default), reject);
        }
      });
    });
  }
});
```

That same code will look as follows after transpiling, only the dynamic imports are updated.
```js

export default Ember.Service.extend({
  someMethodOnThisService() {
    return Ember.RSVP.Promise((resolve, reject) => {
      fetch('/resource').then((data) => {
        if (data.variable === 1) {
          (new Promise(resolve => resolve(require(['app/services/x'])))).then(service => resolve(service.default), reject);
        } else if (data.variable === 2) {
          (new Promise(resolve => resolve(require(['app/services/y'])))).then(service => resolve(service.default), reject);
        } else {
          (new Promise(resolve => resolve(require([`app/services/service-${data.variable}`])))).then(service => resolve(service.default), reject);
        }
      });
    });
  }
});
```

## Using different services based the (mobile) operating systeem

While there is already the concept of dependency injection in Ember, one cannot import different modules in `initializers` or `instance initalizer`. So one has to import all possible modules in an initializer. With `import()` one can decide at run-time which modules to import. 

Now:
```js
import Cordova from 'app/services/systems/cordova';
import Electron from 'app/services/systems/electron';
import BrowserA from 'app/services/systems/browser-a';
import BrowserB from 'app/services/systems/browser-b';

let os = window.navigator.userAgent;
if (os === 'cordova') {
  application.register('service:operating-system', Cordova, {
    singleton: true
  });
}
if (os === 'electron') {
  application.register('service:operating-system', Electron, {
    singleton: true
  });
}
// ... for every operating system an if statement

application.inject('controller', 'opertingSystem', 'service:operating-system');
```

When this RFC is implemented, the programmer use a factory class.

```js
import Factory from 'app/services/systems/factory';

application.register('service:operating-system', Factory, {
  singleton: true
});

application.inject('controller', 'opertingSystem', 'service:operating-system');

// app/services/systems/factory
export default Ember.Service.extend({
  factory() {
    return import(`app/services/systems/${window.navigator.userAgent}`);
  }
});
```

# Future

Now that Ember still concatenates javascript files, this RFC does not effect network performance now. However, if Ember decides to stop concatenating files (when HTTP 2 is available in every setup), this RFC could be extremely useful to import only those modules that are needed.
