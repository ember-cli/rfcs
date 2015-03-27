- Start Date: 2015-03-26
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Allow people to opt-out of some default or use alternatives.

# Motivation

When the commands `new` or `init` are run, some assumptions are done
based on what the community considers best practices, this works great
but there are scenarios where people might not want some of the things
included as defaults.

In a similar fashion to what `Rails` does, we should be able to allow
people to opt-out of some of the things included or replace the
defaults, e.g. allow people to specify SASS.

# Detailed design

The commands `new` and `init` should be extend so they can modify the
generated blueprint based on passed options.

Initially we could support the following options:

- `--skip-packages=[ember-cli-content-security-policy, ember-data, ember-cli-ic-ajax]`
- `--stylesheet sass`
- `--template emblem`
- `--testing karma`

People can save their default in `.ember-cli` and they won't need to
memorize them every time they run `new` or `init`.

Additionally we could introduce the concept of a ".global-ember-cli"
where people can specify default options for all their ember-cli
projects.

# Drawbacks

TBD

# Alternatives

At the moment it seems like this can be done at some level with
blueprints, but I haven't confirmed yet.

# Unresolved questions

Full list of initial options.
