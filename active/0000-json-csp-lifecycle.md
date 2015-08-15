- Start Date: 2015-08-15
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

This request is to add in the ability for addon developers to publish their CSP requirements in a standardised format. Application authors can use this to maintain a valid CSP policy. 

# Motivation

CSP has been implemented wrongly by a lot of addons, mostly this is because of many reasons:

- Lack of education
- Ease of tooling
- Older coding practices

Authors really struggle to implement a CSP policy with the same problems and broken addons.

The changes here should aid in education and tooling for both application authors and addon developers.

Promotes an ecosystem that publishes their expected configuration for security.

# Detailed design

This proposal sits on top of creating a JSON CSP format: [JSON CSP proposal](https://lists.w3.org/Archives/Public/public-webappsec/2015Aug/0073.html)

### CSP.json

In all project routes for applications and addons it will be advised that you have a `CSP.json` otherwise Ember will consider your addonand application to have:
```
{
  "default-src": ["'self'"]
}
```

Integration tests and applications consuming the addon would assume the above setup after this feature landed; this gives the benefit of training addon developers to make sure their CSP.json is correct, which then makes it much easier for the masses to then use the Aden.

Blueprints will need to be adjusted to contain these files.

### Installing/updating a new Adan

Due to the risk of causing security risks by installing an addon, the policy will be checked to see if it is less restrictive than the current compiled policy of the application.
If the policy requires new directives or new items within the values then the installer would warn about the new risks and ask the developer to approve the changes:

    /my/path$ ember install my-plugin
    This addon requires new directives, please confirm if you would like to continue (y/n):
    {"script-src": ["'unsafe-inline'"]}

I suggest the JSON is the full compiled policy using something like [jsondiffpatch](https://github.com/benjamine/jsondiffpatch) to output the differences pre and post addon install comparing the appearance of the compiled JSON only.

### CLI interface

Due to the ability to update an addon through npm perhaps or manually copying code, there is the possibility that the approval error message is never recieved. So Ember will provide new debugging tools to aid in making CSP a first class passenger on Ember.

#### Show

`ember CSP show` will show the raw output for the current application and all of its addons.

This command could be expanded to have adapters for different server implementations too (e.g. `ember CSP show --type=apache`).

#### trace

`ember CSP trace` will show a trace of where each CSP directive came from in your applications:

```
application:
  {"default-src": ["'none'"]}
  ember-table:
  {"style-src": ["'unsafe-inline'"]}
    other-addon:
    {"script-src": ["'unsafe-eval'"]}
```

#### help

`ember CSP help` Will show implementation help for each CSP directive and how it should be limited, what the exploits are etc.

# Drawbacks

Perhaps Ember developers enjoy the pain in development!?

# Alternatives

Originally the proposed discussion was around implementing the addon to use a consolidated file like `package.json` is written to; that post install would merge in the packages rules into the applications ones. However I see this becoming a maintenance nightmare in a larger application as knowing the source of each requirement will become complicated.

I'm considering proposing includes into the CSP.json file itself to the addons, however this doesn't seem to fit with Ember really. The advantage is that I would still know application scope but also reference all my includes also.

```
{
  "directives": {
    "style-src": ['self']
  },
  "includes": [
    "node_modules/some-addon/CSP.json"
  ]
}
``` 

# Unresolved questions

- Do we want a scripted interface?
  - I personally would prefer to add more CLI interfaces to interact with the JSON file than make a scripted interface.
    The main rationale here is that other frameworks picking up the same format reduces the training cost of Ember and increases adoption for the format.
  - I do think this can be deferred to a second iteratation.
- Will this impact build performance looking through all the files then merging the policies each time (Not just a straight JSON merge).
