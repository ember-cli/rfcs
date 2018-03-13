- Start Date: 2018-03-13
- RFC PR: (leave this empty)

# Add a Service Worker with minimal asset caching to a default Ember CLI application
## Summary
This RFC introduces a Service Worker to the default app blueprint. It will only cache the most essential assets, such as `index.html`, and the vendor and app specific JS and CSS files.

## Motivation
A Service Worker allows you to cache assets and other resources in a  way that loading a page becomes reliable and fast, independent of what the network conditions are.

Not all Ember applications are destined to become a Progressive Web App, but that doesn't mean that applications shouldn't leverage a simple Service Worker script to somewhat improve the reliability of loading it.

## Detailed design
### Registration
A Service Worker script named `sw.js` should be compiled and placed at the root level of the build result. The Service Worker script should be registered with a registration script that is inlined in `index.html`. The registered Service Worker should have the `rootURL` that is configured in `config/environment.js` as scope.

### Caching
The default Service Worker configuration should only cache the following files:

* `index.html`
* `assets/vendor.js`
* `assets/<app-name>.js`
* `assets/vendor.css`
* `assets/<app-name>.css`

The cached `index.html` file should be served by the Service Worker whenever a `fetch` event happens that has the request mode of `navigation` and the `accept` header of the request has `text/html` as value.

### Service Worker invalidation
By default each build should include a random sequence of text in the compiled output of `sw.js`, this way the Service Worker gets invalidated after every build.

### Kill switch
The build process should be able to take a flag that acts as a kill switch in case the Service Worker needs to be unregistered in the deployed environment as the result of a bad deploy.

When this flag is given, the registration script for the Service Worker should be altered to immediately unregister the current registered Service Worker.

## How we teach this
The Ember CLI documentation should be updated with a notice that a default Ember CLI application registers a Service Worker and explain the functionality of said Service Worker.

It should also include a warning that a Service Worker must be served over HTTPS and served from the same origin to function correctly.

It might also be worth to add a few pointers on how to set up developer tools to improve the development experience with an active Service Worker.

## Drawbacks
The Service Worker adds a complex layer of caching that might trip up some developers that don't have a lot of experience with Service Workers. However, in our opinion, the upside  of having a Service Worker that improves page loading experience significantly outweighs the cons.

## Alternatives
Instead of adding it by default, it could be added by a generator supplied by Ember CLI.

