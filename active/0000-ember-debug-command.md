- Start Date: 2015-01-04
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

A new command **debug** to dump enough information about an ember-cli app to a file that can be attached to issues.

# Motivation

An issue in the ember-cli ecosystem might be because of multiple reasons,
- Bower & npm dependencies used in the project
- Incorrect brocfile setup
- Config is not right

Its hard for someone to take enough information out of their app and create sample repos to reproduce these issues. Instead they would just run `ember debug`, look through the generated file to make sure nothing super secret is put in to it and submit this file along with their issue. Makes the life of contributors a little more easier since they’d have most of the details they need for initial investigation right there in the generated report.

# Detailed design

This is the bulk of the RFC. Explain the design in enough detail for somebody familiar
with the tool to understand, and for somebody familiar with the implementation to implement.
This should get into specifics and corner-cases, and include examples of how the feature is used.

# Drawbacks

Why should we *not* do this?

# Alternatives

What other designs have been considered? What is the impact of not doing this?

# Unresolved questions

What parts of the design are still TBD?
