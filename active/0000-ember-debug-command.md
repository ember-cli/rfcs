- Start Date: 2015-01-04
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

A new command **debug** to dump enough information about an ember-cli app to a file that can be attached to issues.

# Motivation

An issue in the ember-cli ecosystem might be because of multiple reasons,
- Bower & npm dependencies used in the project
- Incorrect Brocfile setup
- Config is not right

Its hard for someone to take enough information out of their app and create sample repos to reproduce these issues. Instead they would just run `ember debug`, look through the generated file to make sure nothing super secret is put in to it and submit this file along with their issue. Makes the life of contributors a little more easier since they’d have most of the details they need for initial investigation right there in the generated report.

# Detailed design

`ember debug` would gather the following information
	-	ember-cli version
  - npm version
	- node version
	- OS
	- bower dependencies 
	- npm dependencies
  - Brocfile.js
	- config/environment.js
	- Files counts in node_modues, vendor, bower_components, app
	- Output of a command. i.e, sometimes running `ember build` might cause the issue so when running this command users could possibly pass an argument `—command=`(e.g. —command=debug) which we can run and include the output along with the report.

# Unresolved questions


