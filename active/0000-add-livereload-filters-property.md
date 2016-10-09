- Start Date: 2016-10-10
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

This RFC suggests exposing a project property called `liveReloadFilters`. This is an array of
strings or regexes that will be used to filter *out* paths from ember-cli's array of post-build changed filepaths that
are sent by ember-cli to the livereload client in the user's browser. If no paths remain after
filtering, no livereload is triggered.

The existing `liveReloadFilterPatterns` property is somewhat of a misnomer because it doesn't filter anything from the livereload list. Instead, it
is only used to determine [whether a livereload should be triggered at all](https://github.com/ember-cli/ember-cli/blob/16679f3bb2af590d1eb0065f5c50e041788404e3/lib/tasks/server/livereload-server.js#L121-L137), and it uses the changed source file (pre-build) as its input. If the edited source file matches any of the
filter patterns, ember-cli skips the livereload trigger.
If ember-cli determines that a livereload should be triggered (i.e., none of the filter patterns matched), it [diffs the post-build tree against the previous build tree
to generate a list of changed post-build filepaths](https://github.com/ember-cli/ember-cli/blob/16679f3bb2af590d1eb0065f5c50e041788404e3/lib/tasks/server/livereload-server.js#L145-L160). This list is sent to the browser livereload client.

Currently, there is no mechanism in ember-cli to influence which files are sent to livereload. The existing `liveReloadFilterPatterns` property is only used to
determine whether a livereload should happen at all, and only operates on the pre-built source files, not the build output.

This RFC therefore also suggests removing or deprecating the existing `liveReloadFilterPatterns` property because the proposed `liveReloadFilters` property can be used
for all existing use cases along with the new ones described here.

# Motivation

This allows an ember-cli project to effectively mark some build artifacts as "not needing to be sent to the browser livereload client."
For example, for addons like `ember-cli-fastboot` (which generates a `package.json` manifest file) and `ember-cli-amp`
(which generates an alternative index.html file) that create build artifacts that are not web/browser-specific files,
`liveReloadFilters` would allow a user to prevent those from triggering unnecessary livereloads, no matter which source file changes resulted in their being rebuilt.

In the case of the `ember-cli-amp` addon, it generates an alternative `amp-index.html` file with the app's CSS inlined. As a result,
ember-cli rebuilds `amp-index.html` every time the user edits a source css file. The list of changed files that ember-cli sends to the livereload client includes the built
CSS file as well as the built `amp-index.html` file (e.g.: `files: ['my-app.css', 'amp-index.html']`). The livereload client will refresh the
entire page because it detects that both css and non-css (i.e., html) files have changed. If the user were able to mark the "amp-index.html"
as a path to be filtered out, the livereload client would only be notified about the css change and can "hot-refresh" the styles without having to reload
the entire page.

The existing `liveReloadFilterPatterns` property cannot be used for the same use case because it filters on the
file that changed on disk before ember-cli rebuilt. Since the non-web artifacts in question only exist *post-build*, and are not
edited directly by the user, the `liveReloadFilterPatterns` cannot be used to match and ignore them.

This RFC suggests removing or deprecating the existing `liveReloadFilterPatterns` property because the proposed `liveReloadFilters` property
should be sufficient for all existing use cases as well as the new ones described above. In addition, the existing internal tooling that ember-cli
uses to receive file-changed notifications debounces its calls and only sends the first file that was changed when multiple files all change within its
debounce time (see [#6191](https://github.com/ember-cli/ember-cli/issues/6191) and [broccoli-sane-watcher#35](https://github.com/broccolijs/broccoli-sane-watcher/issues/35)). Because of this, the existing livereload functionality can incorrectly fail to trigger a livereload because it is not notified of all the files that changed.
Filtering on the post-build changed files does not have the same issue.

# Detailed design

A project property called `liveReloadFilters` will be added that is an array of strings or regexes, e.g.:
```
liveReloadFilters: [
  'amp-index.html'
]
```

Assuming the user is using the `ember-cli-amp` addon that builds an `amp-index.html` file every time the app's CSS changes,
after the user edits a source CSS file (e.g., `app.css`), ember-cli rebuild. After rebuildling it diffs the new tree with the previous tree and
generates a list of changed files. This list would be:

```
files: ['amp-index.html', 'my-app.css']
```

At this point it will check for the existence of the `liveReloadFilters` property and filter the changed file list, removing every file that either exactly equals the string if the value is a string, or matches the regex if the value is a regex. In this case the string "amp-index.html" matches exactly and is removed, leaving the following list of changed files that ember-cli then sends to livereload:

```
files: ['my-app.css']
```

The livereload client in the browser will then hot-reload the styles rather than doing a full page refresh.

# Drawbacks

If a particular source file change results in many, varied output files, it may be unwieldy to add many `liveReloadFilters` to filter them all out.
In this case, it would be simpler to use the existing `liveReloadFilterPatterns` to mark the source file as "ignorable", so that livereload is never triggered if it changes (regardless of what the build output is).

# Alternatives

Instead of deprecating/removing the existing `liveReloadFilterPatterns` property, it could remain, possibly with a name change to something that indicates that
it operates on the file paths on disk pre-build, and is used to determine whether a livereload should happen at all (e.g., `liveReloadSourceTriggerExclusions`),
in order to provide the ability for a developer to filter out source (pre-build) in addition to post-build file changes.

Another alternative: Rather than adding a new `liveReloadFilters` property, simply reuse the current `liveReloadFilterPatterns` name for the new functionality proposed by this RFC
rather than adding a new, very-similarly-named `liveReloadFilters` property.
This existing property name is a good descriptor of the functionality to be added. The old functionality could simply be removed. The drawbacks to this approach are that it could be confusing
for users, difficult to teach, and could make semver-ing the change in ember-cli awkward.

# Unresolved questions

