- Start Date: 2015-05-19
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Provide a unified API for linting for complete choice of testing framework and linting platform.

# Motivation

Allow user to have full control over what testing language and what linter they prefer.
Currently ember-cli is tied into using JSHint always rather than being fully agnostic.

# Detailed design

In ember-cli addons main file.

Expose a test generator API for the linting addon to call:
```
  lintTree: function(type, tree) {
    return jshintTrees(tree, {
      jshintrcPath: this.jshintrc.tests,
      description: 'JSHint ' + type,
      testGenerator: this.testGenerator
    });
  }
```

Expose a new test Generator hook that would return the lint errors transformed to the test language (This will likely be in the testing framework addon):
```
  testGenerator: function (relativePath, errors) {
    if (errors) {
      errors = "\\n" + this.escapeErrorString(errors);
    } else {
      errors = "";
    }

    return "describe('JSHint - " + relativePath + "', function(){\n" +
      "it('should pass jshint', function() { \n" +
      "  expect(" + !!errors + ", '" + relativePath + " should pass jshint." + errors + "').to.be.ok(); \n" +
      "})});\n";
  }
```

# Drawbacks

- The API for all linters will restrict the API of testGenerator hook.
- Errors in the linter not returning the correct API will cause issues in outputting.

# Alternatives

- Consider using native `new Error` for APIs into testGenerator.
- Consider using object instead of multiple arguments for testGenerator.
- Use blueprints instead of testGenerator.

# Unresolved questions

- What would the best API be for testGenerator?
- How do we make ember handle multiple extensions of testGenerator?
- Should we trigger a fatal error if testGenerator doesn't receive the valid API usage?

