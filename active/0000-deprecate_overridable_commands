- Start Date: 2016-03-21
- RFC PR:
- ember-cli Issue:

# Summary

Deprecate the ability for addons to override existing ember-cli commands to improve their execution time.

# Motivation

While working on ember-cli commands a lot lately i realized, that when a user enters `ember help` or `ember version`, after figuring out the matching built-in command (takes less then 1ms), we look through all modules and nested modules to see if this command is overridden somewhere. This is not only unlikely for most commands, it also takes ~1300ms on a **new** project that only has the standard modules from package.json blueprint. I assume this time will grow with increasing project size.

# Detailed design

Implementing this would be straight forward. We already check for built-in commands before traversing the dependencies. We would only need to early-return once we find a matching built-in command. (I'm happy to provide a PR if we decide on this deprecation)

# Drawbacks

The hole issue is a matter of priorities. I personally feel that speed of ember-cli is more important than generically overridable commands but others could disagree. I have no intuition of how widely this feature is used or if any addon is overriding commands in the first place. But if this is the case, they would need to switch to a probably more verbose command name.

# Alternatives

Alternatively we could try to speed up the module traversal but i guess we will hit a bottle neck on node level there when it tries to parse all those files. Also we could decide only to make a subset of commands non-overridable.

# Unresolved questions

-
