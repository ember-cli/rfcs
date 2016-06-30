- Start Date: 2016-06-30
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Treat template blueprints like test blueprints, they are their own blueprint and
included automatically.

# Motivation

Why are we doing this? What use cases does it support? What is the expected outcome?
I want to override the blueprint for either a route or a component. I want to
make changes to the js file only and not worry about copying the default
template from upstream along with the default replace value logic. Vice-versa
for modifying templates without worrying about the js file.

# Detailed design

We create blueprints for `component-template` and `route-template`. We move the
template blueprints to these folders. These blueprints are automatically called
like the test blueprints because they end in `-template`.

We then either deprecate the `template` blueprint because it is confusing
whether it is a route or component template, or we just leave it. Or we can just
delegate that decision to the legacy blueprint project or ember addon.

Making this a bit more concrete:

MOVE FROM: https://github.com/ember-cli/ember-cli-legacy-blueprints/blob/master/blueprints/component/files/__root__/__templatepath__/__templatename__.hbs
TO: https://github.com/ember-cli/ember-cli-legacy-blueprints/blob/master/blueprints/component-template/files/__root__/__templatepath__/__templatename__.hbs

MOVE FROM: https://github.com/ember-cli/ember-cli-legacy-blueprints/blob/master/blueprints/route/files/__root__/__templatepath__/__templatename__.hbs
TO: https://github.com/ember-cli/ember-cli-legacy-blueprints/blob/master/blueprints/route-template/files/__root__/__templatepath__/__templatename__.hbs

DEPRECATE: https://github.com/ember-cli/ember-cli-legacy-blueprints/tree/master/blueprints/template

And the underlying changes that support that automatic invocation.

# Drawbacks

Added complexity. For those familiar with blueprint structure, not immediately
knowing where the template went.

# Alternatives

We could expand the `template` blueprint to take options specifying whether it's
a route or component template. This wouldn't be consistent to how we do other
blueprints like tests.

If we don't do this, there is more maintenance when upstream blueprint changes
happen to files I'm not overriding for a purpose, only because I had to.

# Unresolved questions
