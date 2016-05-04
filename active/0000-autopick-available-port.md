- Start Date: 2016-05-03
- RFC PR: 
- ember-cli Issue: 

# Summary

Fallback for different default `serve` port if its already in use when custom port isn't defined. Also to keep track of an already running ember app.

# Motivation

This RFC is a followup to [ember-cli/ember-cli#5703](https://github.com/ember-cli/ember-cli/issues/5703). When running multiple ember cli server instances on a machine, the default port would already be in use. But the authors of these apps might not want to define a custom port through the cli `--port` option or the `.ember-cli` file. This will also inform users if the current app is already running on the machine.

# Detailed design

#### Track running instance of an app
- Whenever an app starts running in `serve` mode, the cli will generate a file which we'll use to keep track of the process id and port number of an ember-cli instance for the given app. 
- This file will get removed in the `cleanup` hook to account for process exit.
- In case the user tries to run `ember serve` on an app which is already running, we will use the port number saved in the file to let the user know the entire url where the app is already being served

#### Picking a random port when the default is unavailable
- If port `4200` is already being used, we can find an available port in the unassigned range of 49152–65535. This will be similar to the live-reload-port's existing implementation.
- There will be an extra message printed before we print the url during initial build informing the user that the app was launched in a different port. This can even suggest the availability of the port option in the `.ember-cli` file or through the `--port` option.

# Drawbacks

Users might not pay attention to the build output by default when they run `ember serve` and might just prefer to see a failure to launch than us handling it.

# Alternatives

Users will have to figure out if another process is using the default port vs their ember app already running and proceed to work around it themselves. 
