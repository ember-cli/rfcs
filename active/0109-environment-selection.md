# Environment Selection
* Start Date: 2017-07-24
* RFC PR: (leave this empty)

# Summary

The goal of this RFC is to justify and solidify the mechanism that ember-cli uses to determine the environment that any given command will be ran under. 

# Motivation

We have received feedback from our direct users, developer relations folks, and benchmark maintainers suggesting that our current system for selecting the environment does not set our users up for success.

This RFC aims to ensure that users of ember-cli get the correct environment when specified (via `NODE_ENV`, `EMBER_ENV`, or `--environment` command line flags) and when using the default.  

# Terminology

* Command line specified environment -- When the command line invocation includes a specified `environment`  (either `--environment=xyz` or uses the environment shortcut aliases (e.g. `-prod` or `-dev`).
* ember-cli specific process environment variable specified environment -- When the shell that invokes the command has `process.env.EMBER_ENV` set.
* General purpose process environment variable specified environment -- When the shell that invokes the command has `process.env.NODE_ENV` set.
* Test command -- A command used to run tests (e.g. `ember test` or `ember test --server`).
* Interactive command -- A command which is not intended to "complete". These commands commonly perform rebuilds and/or re-run tests (i.e. `ember build --watch`, `ember serve`). 
* Non-interactive command -- A command intended to be ran to completion (e.g. `ember build` or `ember test`).

# Detailed design

The following mechanism will be used to select the environment by each ember-cli command (in order):

1. Command line specified environment (e.g. `ember serve --environment production`)
2. ember-cli specific process environment variable specified environment (e.g. `EMBER_ENV=production ember serve`)
3.  General purpose process environment variable specified environment (e.g. `NODE_ENV=production ember serve`)
4. No environment was specified, default based on command type:
	* Test commands -- Default to `test` environment.
	* Interactive commands -- Default to `development` environment.
	* Non-interactive commands -- Default to `production` environment.

A pseudo-code based example:

```javascript
function isInteractiveCommand(commandName, knownArgs) {
  switch (commandName) {
  case 'test':
    return knownArgs.includes('server');
  case 'build':
    return knownArgs.includes('watch');
  case 'server':
    return true;
  default:
    return false;
  }
}

function determineEnvironment(commandName, args) {
  let knownArgs = Object.keys(args);

  // handle command line specified environment
  if (knownArgs.includes('environment')) {
    return args.environment;
  }

  // handle ember-cli specific process environment variable
  if (process.env.EMBER_ENV !== undefined) {
    return process.env.EMBER_ENV;
  }

  // handle general purpose process environment variable
  if (process.env.NODE_ENV !== undefined) {
    return process.env.NODE_ENV;
  }

  if (commandName === 'test') {
    return 'test';
  }

  return isInteractiveCommand(commandName, knownArgs);
}
```

This detection will be done early in the command process, and the `process.env.EMBER_ENV` environment variable will be set with the selected environment (so that addons and other tools are able to know the environment being used).

# How We Teach This

We will need to ensure that the descriptions of various commands are updated to ensure that the default environment is properly annotated/updated, and that any guides referencing "environment"s on ember-cli.com or guides.emberjs.com are updated.

# Drawbacks

Changing anything around default environment selection has the possibility of breaking some use cases that may have relied on the current default. Note: I believe this to be a pretty unlikely scenario, but it seems reasonable to list/discuss.

# Alternatives

We could do nothing...
