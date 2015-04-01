- Start Date: 2015-03-31
- RFC PR:
- ember-cli Issue:

# Summary

When an add-on provides a blueprint that has the same name as a default Ember-CLI blueprint, it overrides it. You can see that the results of `ember generate --help` now lists the overridden blueprint instead of the default one. It would be nice to be able to force an overridden blueprint to run.

# Motivation

I'm in the process of migrating my global Ember application to Ember-CLI. My legacy codebase is written in CoffeeScript, and I don't have time to port all complex legacy code over to ES2015-style Javascript. However, I want to move over to JavaScript eventually, so it's nice to start writing ES2015 JavaScript when converting legacy CoffeeScript files that have less complicated logic.

In my case, `ember-cli` and `ember-cli-coffeescript` both provide blueprints for the base generators (mixin, route, controller, etc.). As we migrate our old Ember Global App over to Ember CLI, it's handy to be able to `ember g route` and have`ember-cli-coffeescript` handle the blueprint (as it does when installed) and copy over the existing logic from our old app.

However, there are new features and tests that I'd like to start developing in JavaScript. As implemented today, since `ember-cli-coffeescript` overwrites the default ember-cli blueprints, I have to [flip a switch](https://github.com/kimroen/ember-cli-coffeescript/tree/v0.10.0#blueprints) every time in my configuration, which [short-circuits](https://github.com/kimroen/ember-cli-coffeescript/blob/v0.10.0/index.js#L26) the `blueprintsPath` in `ember-cli-coffeescript` and makes default ember-cli blueprints available (since the add-on is no longer overriding the blueprint definition) and will result in a generated JavaScript file.

# Detailed design

What I'm proposing is having an easier way to run a different add-on's blueprint from the command line. For instance, instead of flipping the switch in the configuration file, it'd be great if I could leave `ember-cli-coffeescript` add-on in tact and tell `ember generate` that I want to run the blueprint from the base `ember-cli` package, instead of `ember-cli-coffeescript`'s overridden version.

I imagine this would be something like
```
ember generate ember-cli:route
```

This would force the `ember-cli` route blueprint to execute, instead of the overridden `ember-cli-coffeescript` route blueprint. You could provide the name of any installed Ember CLI add-on and the name of a blueprint it defines using the same format.

I think that, even with this change, the logic that exists today should still be the default. When I call `ember g route`, the add-on should still be run as the default. However, this additional functionality is really handy when in a situation like I described above. Lots of add-ons override provided blueprints, so I imagine this could be useful for other use cases, as well.

# Drawbacks

Since this RFC only adds optional functionality and does not change the current logic, I don't see obvious drawbacks.

# Alternatives

The alternative would be to continue relying on short-circuting the blueprint path in the add-on's `index.js` file. However, this clunkier than being able to specify in one command that you want a different blueprint to run, especially if you have two add-ons overriding a provided blueprint. It is also easier because add-ons wouldn't have to implement reading the configuration file and short-circuiting in their `index.js` file if a flag is set.

# Unresolved questions

The only unresolved question, in my mind, is how exactly it would be implemented. There appears to be an [`overridden` flag](https://github.com/ember-cli/ember-cli/blob/master/tests/unit/models/blueprint-test.js#L168) in the blueprint test, but I'm not extremely familiar with how the actual generate command works internally.
