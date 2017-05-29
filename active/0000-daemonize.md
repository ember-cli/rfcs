- Start Date: 2015-05-26
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

ember-cli will run as a daemon in the background, rather than being started for each `ember` command.

# Motivation

Currently running an `ember` command can take quite some time, especially when a lot of addons are installed.
This is mainly due to the "boot up" period, when Node.js / io.js loads ember-cli itself and *all* the addons.
Node.js / io.js unnecessarily compiles / loads a huge amount of files that very likely aren't relevant to the task.

# Detailed design

Instead of rebooting ember-cli everytime an `ember` command is executed, there would be a service daemon running in the background
that performs the tasks. The major advantage is that his daemon would only have to load *once* and could run all subsequent
tasks in an instant.

The `ember` binary is merely interface to the daemon. It's sole task is proxying the entered task to the daemon,
which then in turn does all the heavy lifting.

# Drawbacks

##### Yet another background service

However, this isn't a backgroudn service that auto starts on system boot, but only when it is invoked by the user.

##### Bigger risk of memory leaks

Memory leaks would add up over time.

##### Different UX

Users would have to learn how to and get accustomed to start and stop the daemon, depending on how ths is implemented.

However, users generally run `ember serve` and keep it running in a different terminal in the background.
They should be used to having `ember` run in the background.

# Alternatives

##### Change the way addons are loaded

The addon loader and addon blueprint could be changed in a way that would allow addons to export their commands and blueprints,
without having to load the whole addon. AFAIK an addon is currently fully loaded even if a) it doesn't need most of
the functionailty for the given task (`ember delete`) or b) it isn't even relevant for the given task (like `ember version`).

The major drawback would be that all addons would have to be rewritten and that there still is a boot up period, although it'd be shorter.

# Unresolved questions

##### How to actually implement this?

Sadly I don't know much about the technical intricacies of ember-cli, but I could imagine, that for the surface API for addons
not much (if not nothing) would have to change.

##### How would the background service be invoked by the user? How and when would it be shut down?

I see two possibilities here.

- Automatically start up (if not already running), whenever an `ember` command is invoked.
  Shut down automatically after x minutes of user inactivity.
  Also shutdown, when the user runs `ember shutdown`, or something similar.
  
- Start up with the command `ember daemon` (or similar) and *don't fork to the background but keep running in the foreground*.
  This would be very much like `ember serve`. The user would close the daemon with `Ctrl + C`.
  The user could also easily see the log output, by just switching to the terminal.

##### How would the cli connect to the background service?

On Unix systems that could be easily done with Unix sockets. The daemon would create a hidden `.ember-cli.sock` file
in the root directory of the project (name irrelevant). This file serves as a [Unix socket](https://nodejs.org/api/net.html#net_net_createserver_options_connectionlistener).

However, there is no such thing as Unix sockets on Windows machines. In this case, I would recommend starting the server
on a random port and writing that port number to `.ember-cli.port` (name irrelevant).
