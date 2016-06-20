- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Provide `GET /_testem/lcov` end-point, enabling test runners such as `ember-exam` and code coverage tools such as `ember-cli-blanket` to work well together.

# Motivation

Code coverage is an important tool in ensuring product quality. With large code-bases, running tests in parallel is important to ensure fast feedback cycles. Unfortunately combining these two (and likely other things) without first-class support in ember-cli is problematic. Rather then implementing an ad-hoc protocol between these two popular add-ons, ember-cli could provide an add-on agnostic `lcov` endpoint.

This RFC also attempts to be test framework independent, runner and code coverage tool independent, merely enabling the above but not coupling to either `ember-cli-blanket` or `ember-exam`

# Detailed Design

The current thought is that this will largely be implemented in `testem` with the exception of the configuration options. It could be implemented entirely in `ember-cli` but this seems like a primitive testem could support nicely

### A New End Point

The following end-point will be provided by ember-cli (potentially via testem)

``` js
POST /_testem/lcov
params:
  lcov: <lcov data>
During any given test run, ember-cli will:
```
For a given test run all `POSTS` to `/_testem/lcov` will include `lcov` data. It will be stored in memory and once the test run completes, all submissions will be merged and written to `options.lcov.outputdir` which will default to `/coverage/`. If no `POST` occurs, no `/coverage/lcov.info` will be written.

By merging all `POSTS` during a given test run, both `ember test` and `ember exam --split` & `testem.parallel: x` should work.

To ensure `POSTS` from the current run is processed, we should include some *test session* identifier. This will ensure the `lcov` data wont be tainted by other test runs.

# Alternatives

- could be implemented in an additional add-on
- could wrap `ember exam` and script the merge before/after run

# Unresolved Questions

- is `lcov` sufficiently agnostic to "just work" ?
- how do store the session identifier
- should we automatically run `genhtml` to produce high fedility output in `/coverage`?

# Prior Art

- https://github.com/gotwarlost/istanbul-middleware/ (investigating if we can use this, or some of this)

New Relic
