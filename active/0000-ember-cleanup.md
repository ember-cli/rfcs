- Start Date: 2015-03-31
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Allow people to remove redundant files for reduced complexity of application and less byte size.

# Motivation

Why are we doing this? What use cases does it support? What is the expected outcome?

When I build my app using `ember-cli` I often generate, for example, `routes`. Then not to duplicate my `template` code, I use `templateName`. When I do this multiple times I may forget that I leave `templates` generated just with `{{outlet}}` as their content.

Also, after some time of developing an application I have `controllers` that just extend `Ember.Controller` and nothing more.

I think both default `templates` and default `controllers` aren't needed in my app's source. Also, one might forget to clean some imports in files which are no longer needed.

I believe if `ember-cli` had a command to just cleanup all these things, it would be easier to maintain application.

Potential benefits:
  - reducing number of files
  - reducing lines in files
  - reducing confusion
  - reducing byte size

# Detailed design

Create `ember cleanup` command. It could have possible keywords used with it(example `ember cleanup imports`):

### imports
It should delete all not used imports in files.

Design:
- get list of all `app/*` files
- loop through all files
  - save list of imports to array where `key` is the name of import(for example, `import Ember from 'ember'` - name is `Ember`)
  - go through each line and check if it has one of `keys` from imports array, if yes delete found `key` from array, else do nothing
  - when file is processed to the end check if initial imports array is empty, if yes do nothing, if not delete lines with imports that are not used

---

### controllers
Deletes controllers with default content(just extending from `Ember.Controller`).

Design:
- get list of all files from `app/controllers/*`
- iterate over each file and check if has only:
 - `import Ember from 'ember'`
 - `export default Controller = Ember.Controller.extend()`
- if yes then delete file because there should be no difference between programatically auto-generated controller at runtime and this file

---

### templates
Deletes templates with just `{{outlet}}` as their content.

Design:
- get list of files from `app/templates/*`
- iterate over them
- if file contains only `{{outlet}}` then delete it

---

### components
Same pattern as `templates` could be followed for `components` except for `{{yield}}` instead of `{{outlet}}`.

---

### routes
Same pattern as `controllers` could be followed for `routes` except for `Ember.Route.extend()` instead of `Ember.Controller.extend()`.

---

### unused
Check for no longer referenced components, helpers, templates, controllers and eventually delete them.

Design:
##### helpers
- get list of helpers from `app/helpers/*`
- search for using helpers in templates as `{{helper}}` or `{{#helper}}`
- if any occurence is found for concrete helper then don't delete it, else do it

#### components
Same pattern as with helpers but also check JavaScript files is any file imports this component(for example to extend from it).

Design like:
- Search for `/components/component-name` if anything is found then don't delete it component.
- Else search for `./component-name` and check if file that has `./` import is in `app/components` directory, if yes, don't delete it, else do it.

#### controllers
- could be searched for `needs: ['controller-name']`
- search for: `controller: 'controller-name'`
- check if controller file path can be linked with any route from `router`
- check if `controller` file is imported in any file
- if no occurences found, delete file

#### templates
- check if it is used by any route, component
- search for `templateName: 'template-name'`

------------------
At the end of each command summary should be presented to users with list of full paths of files that were deleted or affected by deleting imports.


# Drawbacks

I remember sometimes even empty controller file was needed for my application or default outlet file. I don't know if this is still an issue.

# Alternatives

Ember Doctor could be alternative but I think its purpose is to "heal" application not to "clean" it.

# Unresolved questions

- Could following `cleanups` be implemented?
 - check for unnecessary bower(I know it's going to be deprecated, so maybe some replacement for bower), npm packages - this is similiar to ember doctor concept but it would be possible to delete dependencies programatically

 - check for CSS/SCSS/other which is not used by application code anymore and eventually remove it

 - check for class names not being dasherized and eventually rename it both in .css and templates, js files
