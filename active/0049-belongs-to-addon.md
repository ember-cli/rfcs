- Start Date: 2016-04-05
- RFC PR: #49

# Summary

Add a hook that allows Ember Addons to know whether they're being included in an app or inside of another addon.

# Motivation

Some addons may need to do extra or less tasks when being the child of an addon versus an app. Currently there is not a documented or prescribed way to determine if an addon's parent object is an app or another addon. Since there are varying ways to determine this, it seems like a good idea to provide a standard method to call that resolves this question.

# Detailed design

We could expose a method on `EmberAddon` that returns a boolean that indicates whether its parent is an app or an addon.

```javascript
belongsToAddon: function() {
  return flag; //... true if the parent is an addon, false otherwise ...
}
```

Alternatively it could return a string, more or less a kind of enumeration, that describes the parent's type.

```javascript
parentType: function() {
  if (parent type is an app) {
    return 'app';
  } else {
    return 'addon';
  }
}
```

# Drawbacks

This may be a slight edge case for Ember addons out in the wild. However, internally at my company there are addons that include other addons frequently, which makes this use case very common for me.

# Alternatives

Do nothing. Let everyone determine their parent type for themselves.
