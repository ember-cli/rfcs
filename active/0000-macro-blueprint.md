# macro blueprint RFC

- Start Date: 2016-09-12
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Add a `macro` blueprint to the default list of blueprints.

# Motivation

To make it slightly easier for people to make custom computed macros. The syntax of a custom macro is already dead simple, but having a default structure, starting point, and folder location would help beginners get started.

# Detailed design

The new blueprint would be added to `ember-cli-legacy-blueprints` since the ember addon is not ready yet. It would generate a file that looks like this:

``` js
import Ember from 'ember';

export default function(/* keys... */) {
  return Ember.computed(/* keys... */, function() {
    // your normal computed code
  });
}
```
Then once filled in, could be used like:

``` js
import myMacro from 'my-project/macros/my-macro';

//...

myValue: myMacro('key1', 'key2')
```
# Drawbacks

The code to structure a computed macro is so simple, that it may be unnecessary to turn it into a blueprint. Although I would argue that beginners may not know how simple it is. It may seem like a daunting task to start making your own macros. This could help start them off.

# Alternatives

There's no other design to consider. The impact of not doing this is custom computed macros continues to be a little mystified and underused.

# Unresolved questions

The final structure of the blueprint can be debated. I vote for lots of layman's comments to tell people where they could put things for more complicated macros. Similar to our "ember-cli-build.js" file.
