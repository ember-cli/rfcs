- Start Date: 2016-01-18
- RFC PR:
- ember-cli Issue:

# Summary

Add an additional(and optional) pod directory for grouping ember-data resources together.

# Motivation

For large Ember applications, grouping ember-data files together by filetype creates the same growing pains that using a pod format works to alleviate. There can be a lot of different models that are not neccissarily tied to just one route, or they are tied to many routes, and as such, it most often does not make sense to place those files inside of a pod directory. If the application is communicating with multiple api's, or if you are communicating with a single api that is inconsistent across endpoints, you will wind up with many different adapters and serializers to go with your models. These files are very much related, and the developer experience is made easier by keeping them together.

# Detailed design

Given a typical pod based ember app

```
app
  |---pods
  |   |
  |   |---index
  |   |   |---route.js
  |   |   |---controller.js
  |   |   |---template.hbs
  |   |
  |   |---posts
  |       |---route.js
  |       |---controller.js
  |       |---template.hbs
  |       |
  |       |---comments
  |           |---route.js
  |           |---controller.js
  |           |---template.js
  |
  |---services
  |   |---...
  |
  |---components
  |   |---...
  |
  |---helpers
  |   |---...
  |
  |---mixins
  |   |---...
  |
  |---models
  |   |---user.js
  |   |---post.js
  |   |---comment.js
  |
  |---adapters
  |   |---user.js
  |   |---post.js
  |   |---comment.js
  |
  |---serializers
  |   |---user.js
  |   |---post.js
  |   |---comment.js
```
You can see two seperate logical groupings in action. The pods have their related resources, but the same issue still exists with the ember-data resources. At this point a developer has intentionally stated their interest in sorting the application by logical groupings, but is only able to take the effort partway without a significant ammount of work.


Having the ability to optinally group ember data resources together, it helps to facilitate grouping the logical peices of code together, and makes the code base feel more unified in it's structure.

```
app
  |---pods
  |   |
  |   |---index
  |   |   |---route.js
  |   |   |---controller.js
  |   |   |---template.hbs
  |   |
  |   |---posts
  |       |---route.js
  |       |---controller.js
  |       |---template.hbs
  |       |
  |       |---comments
  |           |---route.js
  |           |---controller.js
  |           |---template.js
  |
  |---services
  |   |---...
  |
  |---components
  |   |---...
  |
  |---helpers
  |   |---...
  |
  |---mixins
  |   |---...
  |
  |---models
      |
      |---user
      |   |---adapter.js
      |   |---model.js
      |   |---serializer.js
      |
      |---post
      |   |---adapter.js
      |   |---model.js
      |   |---serializer.js
      |
      |---comment.js


```

It is also more immediately apparent what files are related when looking at the directory structure, and can aid in onboarding a new member of the team to understanding how the specific Ember application works.

Ember-cli invokation could be

```
$ ember g resource --pods user
```

which could create

```
  |
  |---models
      |
      |---user
          |---adapter.js
          |---model.js
          |---serializer.js

```

while

```
$ ember g resource user
```

would result in

```
  |
  |---models
  |   |
  |   |---user.js
  |
  |---adapter
  |   |
  |   |---user.js
  |
  |---serializer
      |
      |---user.js

```


# Drawbacks

Some ember-data objects can be linked to route resources. i.e.

```
app
  |---pods
  |   |
  |   |---index
  |   |   |---route.js
  |   |   |---controller.js
  |   |   |---template.hbs
  |   |
  |   |---posts
  |       |---route.js
  |       |---controller.js
  |       |---template.hbs
  |       |
  |       |---comments
  |           |---route.js
  |           |---controller.js
  |           |---template.js
  |
  |---services
  |   |---...
  |
  |---components
  |   |---...
  |
  |---helpers
  |   |---...
  |
  |---mixins
  |   |---...
  |
  |---adapters
  |   |
  |   |---application.js
  |
  |---models
      |
      |---user
      |   |---adapter.js
      |   |---model.js
      |   |---serializer.js
      |
      |---post
      |   |---adapter.js
      |   |---model.js
      |   |---serializer.js
      |
      |---comment.js


```
which does not always fit into the pod grouping style of applications.

Also, having too many alternate styles to organize the code base may lead to developer confusion.

# Alternatives

We could also use a seperate key for ember-data resources, i.e.

```
app
  |---pods
  |   |
  |   |---index
  |   |   |---route.js
  |   |   |---controller.js
  |   |   |---template.hbs
  |   |
  |   |---posts
  |       |---route.js
  |       |---controller.js
  |       |---template.hbs
  |       |
  |       |---comments
  |           |---route.js
  |           |---controller.js
  |           |---template.js
  |
  |---services
  |   |---...
  |
  |---components
  |   |---...
  |
  |---helpers
  |   |---...
  |
  |---mixins
  |   |---...
  |
  |---adapters
  |   |
  |   |---application.js
  |
  |---data
  |   |
  |   |---user
  |   |   |---adapter.js
  |   |   |---model.js
  |   |   |---serializer.js
  |   |
  |   |---post
  |       |---adapter.js
  |       |---model.js
  |       |---serializer.js
  |
  |---models
  |   |
  |   |---comment.js


```

or assume ember-data resources that are namespaced to a route are acceptable to place in a pod

```
app
  |---pods
  |   |
  |   |---application
  |   |   |---route.js
  |   |   |---controller.js
  |   |   |---template.hbs
  |   |   |---adapter.js
  |   |
  |   |---index
  |   |   |---route.js
  |   |   |---controller.js
  |   |   |---template.hbs
  |   |
  |   |---posts
  |       |---route.js
  |       |---controller.js
  |       |---template.hbs
  |       |
  |       |---comments
  |           |---route.js
  |           |---controller.js
  |           |---template.js
  |
  |---services
  |   |---...
  |
  |---components
  |   |---...
  |
  |---helpers
  |   |---...
  |
  |---mixins
  |   |---...
  |
  |---data
  |   |
  |   |---user
  |   |   |---adapter.js
  |   |   |---model.js
  |   |   |---serializer.js
  |   |
  |   |---post
  |       |---adapter.js
  |       |---model.js
  |       |---serializer.js
  |
  |---models
  |   |
  |   |---comment.js


```

# Unresolved questions

Naming conventions along with the usual "should we even" are still TBD.

Also, any ember-cli syntax for generating the resources from the command line are not completely fleshed out, and are potentially out of scope for ember-cli as they seem to be more related to the ember-data plugin.
