- Start Date: 2015-04-26
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Generate a custom application from a template, bypass (or enhance) the
default application blueprint.

# Motivation

To share common boilerplate application setups.

# Detailed design

Similar to the [Ruby on Rails application
template](http://guides.rubyonrails.org/rails_application_templates.html) feature. Ember-cli and the Ember community could benefit
from companies/individuals creating boilerplate applications to share
via templates. To be run from a template:

```
# local template
ember new foo -t /local/path/to/file.json

# remote template
ember new foo -t http://remote.path/to/file.json

# maybe even gist support
ember new foo -t http://gist.github.com/foo/1235
```

The json file should could contain post application generation
rules.

The rules could take the form of

* code insertion into existing files
* code removal from existing files
* file creation & deletion
* directory creation & deletion
* command line generators to run

`npm` and `bower` would be held off until *after* the
template was applied.
