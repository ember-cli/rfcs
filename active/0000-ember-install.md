- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)

# Summary

`ember install` without arguments should install all packages.
`ember install package-name` should not error if the package is not an
ember addon.

# Motivation

Since Ember CLI can now decide for you whether to use Yarn or not, we would like to
take advantage of that in more ways.

# Detailed design

This is now possible since we don't have to worry about Bower.
Instead of you knowing whether you should run `npm install` or `yarn`,
`ember install` would do that for you.
Since Ember CLI would now potentially be installing non-ember addons
when it wasn't before, it makes sense to expand this to
`ember install package-name` and not error if the package is not an
ember addon.

# How We Teach This

This can be adding to the guides in the project setup and installing addons sections.

# Drawbacks

I don't think there are any reasons not to do this.

# Alternatives

None
