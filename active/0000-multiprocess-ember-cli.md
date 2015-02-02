- Start Date: 2015-02-02
- RFC PR: 
- ember-cli Issue: 

# Summary

Refactor ember-cli to internally use multiple processes and expose and API to allow addons boot processes.

# Motivation

Currently, we run `ember serve` command as a single process. In this process, we manage file watcher, `ember serve`'s Express server and Testem. We are extending this single process with **/serve** directory watcher restarts the Express server when files change. 

We provide **serverMiddlware** hook to allow users to add their proxies to the Express server. Users are interpreting this as a green flag to customize it in any way. Lacking a recommended path for adding API endpoints to their Express's server, users are adding them as Express middleware and using `ember serve` to boot their APIs.

**serverMiddleware** should only be used to add middleware that pertains to serving assets and for configuring proxies. Proxies should be used to mount API endpoints on to the Express App. A middleware should proxy requests to a port that's served by another process. This is the recommended path but we provide no direction for people to implement this. 

By making it easier to boot multiple processes, we can simplify ember-cli's internal implementation and give a happy path for users to boot their APIs in development.

This refactoring also paves the way for us to simplify deploying to production applications that use Ember Fastboot. By allowing Ember Addons to provide information about processes, we'll be able to generate Profile and (with additional work) proxy configurations that would make deployment to container environments like Dokku & Docker a single step process.
 
# Detailed design

## Internal Refactoring

We introduce a new dependency, something similar to [node-foreman](https://github.com/strongloop/node-foreman) and use it to build an internal API for managing processes. Internally, we will refactor the Build Process, Express Server & Testem Server to use this new API to run in seperate processes. This will allow us to restart these processes at will.

## New API

We will expose a **startProcess** that will allow user to return an array of process definitions to be booted.

```
module.exports = {
	name: 'our-api',
	startProcess: function(options) {
		return [
			{
				name: 'our-api-posts',
				command: 'node api-server.js',
				env: {
					foo: bar
				}
			}
		]
	}
}
```

The processes will be automatically stoped when ember-cli is stopped.

# Drawbacks

I don't know.

# Alternatives

Please, suggest.

# Unresolved questions

1. What should API hook be called?
2. Do we need a stopProcess hook? 
3. What other parts of *ember-cli* should run in a separate process?

