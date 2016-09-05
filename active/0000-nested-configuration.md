- Start Date: 2016-10-05
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Provides a unified system for configuring nested engines and addons from their
hosts. This system aims to be flexible enough to enable different configurations
per app/engine, while still being simple and manageable at scale.

# Motivation
Currently, the method for configuring an Application or Engine (a Host) is
directly through modules from the `/config/environment.js` file. Each Host only
has access to its own environment file, and there is no way for a Host to
configure its children. This means that in order for an Engine to have the same
configuration as its Host, all of the environment properties must be manually
duplicated from the parent into the engine. As applications grow in size and more
parts get broken out into engines, this inflexibility can make apps harder to maintain.

Furthermore, there is no equivalent for Addons as their environment file
doesn't get compiled into the final application. There are some community-based
solutions which inject the root application's environment config into the addon,
but this results in tight coupling and should be avoided if possible.

Design constraints and considerations for the unified system:

1. It should statically compile all configurations at build time. This will help
to prevent strange bugs due to changing configurations, and will result in
better overall performance.

1. It should continue to allow engines to have their own configuration.

1. It should allow engines to specify an external API of overrideable
configuration variables.

1. It should allow hosts to specify different configurations for engines based
on mount point.

1. It should NOT propagate configuration variables from parents to children
unless it is explicitly specified that they be propagated.

1. It should allow addons to have configuration variables.

1. It should NOT allow conflicting addon configurations within a single
container. This is a limitation of addons at the moment, as they do not have
their own containers and so only one instance of each version of an addon will
be able to exist in a given host.

1. It should be easy to visually grep and find configuration values.

1. It should be possible to easily view the final output of any configuration
file for a given App, Engine, or Addon.

# Detailed design

This RFC attempts to address the major design points of configuration

1. The API for consuming configurations

1. The API for specifying configurations

1. The process for statically building configurations

We will start with how users will consume configurations in Hosts and Addons
and move on to how they will be specified and built.

## Consuming Configurations

There are two main APIs we can provide for users to consume their
configurations:

1. Using ES6 modules

1. Using the container

The first option implies that we would statically compile a file for each Host
and place it at a special location within the hosts namespace. This is
undesirable for a few reasons:

1. It is orthogonal to the way that ES6 modules work as a whole. When looking
through a list of import statements, each one implies that an actual file exists
at the specified location, and that to look it up all the user needs to do is
load the file. Configurations will not work this way - they will be compiled at
build time and no file or object will exist until then. This will be confusing
to Ember users who aren't familiar with the build system, and new users.

1. It prevents us from providing different configurations on a per-mount-point
basis. Engines can be mounted in multiple locations, but each instance loads the
same modules. The only differentiating factor between the same engine at
different mount points is that they each have a separate container, meaning we
can inject a different configuration object into each.

Given these limitations of modules, the primary public API that this RFC
specifies is a new `inject.config` method on the Ember namespace which would be
used to inject all configuration variables. In the following example, the new
method is compared against the current one.

Current:

```js
import Ember from 'ember';
import ENV from '../config/environment';

export default Ember.Component.extend({
  value: ENV.value
});
```

New:

```js
import Ember from 'ember';

const { inject } = Ember;

export default Ember.Component.extend({
  value: inject.config('my-app.environment.value')
});
```

This function will pull values from a POJO that is compiled at build time and
will be available on the container.

It is important to note that the name of the application is required when
accessing values on the configuration. This is to provide encapsulation for
child addons of the container, which will be using the same API on the same
container and thus will need some other way of differentiating between the
Host's configuration and the addons. We will dive more deeply into the way these
values are constructed in the build section.

## Specifying Configurations

When we are dealing with large applications with many dependencies, we end up
with a dependency tree from App to Engines to Addons. This tree can get very
confusing very quickly, especially in larger applications, so it is important
for us to make sure that we follow the principle of encapsulation - Each node of
the dependency tree should be fully understandable in isolation. We should be
able to see the entry point of every value from the parent, and the exit point
for every value sent to a child, without leaving the code for that node.

Unfortunately, as each of these three types of nodes is different, there will be
some differences in the APIs. We will go through each of the types individually
and highlight the similarities and differences for each one.

### A quick note on Pull-based vs Push-based configurations

We can either conceptualize our configurations of children as being "pulled" in
from the parent node, or "pushed" down to the children. Each style has its
advantages and disadvantages, and the corresponding implementations may be more
or less difficult to build.

Pull based configurations imply:

1. Children have access to the parent container in some way. This could make
scoping more difficult and harder to reason around, and may confuse users who
expect to be able to access arbitrary values on the parent but cannot.

1. Configurations for each node will live both on the node and partially on its
parent. From a build perspective this will be confusing, though from an API
perspective it may make sense.

1. Children cannot operate without a parent, or must first attempt to lookup
values on a parent and then if they do not exist look them up locally. This two
step lookup could potentially be confusing.

Push based configurations imply:

1. Parents will have the ability to overwrite certain portions of their
children's configuration. This could become confusing, as it would not be as
explicit that certain values may be overwritten.

Given these points, this RFC has decided to use the Push model for passing on
configurations.

# Applications

Application configuration will remain very similar to how it stands today. Files
that exist in the `config` directory, such as `config/environment.js`, will be
evaluated and their export will be added to the injectable POJO on a key that
corresponds to the app name and file path.

For instance, given the following application structure

```
my-app/
  configs/
    environment.js
    other-environment.js
    other-folder/
      some-other-environment.js
```

We would produce this POJO

```
{
  "my-app": {
    "environment": {
      //...
    },
    "other-environment": {
      //...
    },
    "other-folder": {
      "some-other-environment": {
        //...
      }
    }
  }
}
```

This POJO would have all of the values exported from each file at the respective
point in the tree, and would be accessed using the `inject` API as specified in
the previous section.

## Engine Configurations

Engine configurations are created in the exact same way as App configs, with the
exception of a newly added file: `config/engine.js`. This file is the basis for
a public API for the Engine which can be overriden by the Host (another Engine
or App).

Host's MAY have a `config/engines/{{engine-name}}.js` file which will override
any of the values specified in the Engine's `engine.js` file. This file MUST
have a default export that is a POJO which will be merged with the engine's
`config/engine.js` file.

This Host's engine config:

```js
// /my-app/config/engines/my-engine.js

export default {
  theme: 'red'
};
```

Combined with this Engine's `engine` file:

```js
// /my-engine/config/engine.js

export default {
  theme: 'blue',
  defaultTimeout: 5000
}
```

Results in this config object:

```js
{
  "my-engine": {
    "engine": {
      "theme": "red",
      "defaultTimeout": 5000
    }
  }
}
```

Which is injected into the engine's container.

For engines which are mounted at multiple locations, the default export will be
applied to each of these mount points. However, the Host config file MAY also
export another object named `mounts`. This object MUST consist of a mapping from
1 or more of the routes the engine is mounted at to an object containing the
configuration values for that engine at that mount point.

This Host's engine config:

```js
// /my-app/config/engines/my-engine.js

export default {
  theme: 'red'
};

export const mounts = {
  "/route/to/my-engine": {
    theme: 'green',
    defaultTimeout: 2000
  }
};
```

Combined with this Engine's `engine` file:

```js
// /my-engine/config/engine.js

export default {
  theme: 'blue',
  defaultTimeout: 5000
}
```

Results in this config object:

```js
{
  "my-engine": {
    "engine": {
      "theme": "green",
      "defaultTimeout": 2000
    }
  }
}
```

Which is ONLY injected into the container of the engine mounted at
`/route/to/my-engine`. The same engine, mounted anywhere else, will receive the
object from the first example.

### Addon Configuration

Like Engines, Addons will add a new file at `config/addon.js` which will provide
the default values and public API for the addon, and like Engines this file MAY
be overriden by its parent with a file placed in
`config/addons/{{addon-name}}.js`. However, unlike Engines, Addons do not have
their own container. This means that there are some additional constraints,
particularly with regards to nested addons.

Addons OR Hosts MAY configure child Addons by placing a file in the
`config/addons` folder. This configuration will be "flattened" from the highest
parent, which will be a Host, all the way to the Addon being configured.

This Host's addon config:

```js
// /my-app/config/addons/my-addon.js

export default {
  foo: 123
};
```

Combined with this Addon's `addon` file:

```js
// /my-addon/config/addon.js

export default {
  foo: 456
}
```

Results in this config object:

```js
{
  "my-app": {
    //...
  },
  "my-addon": {
    foo: 123
  }
}
```

Addons are not guaranteed to only be included once within the dependency graph
of a given Host. Because of this, two Addons may include the same child Addon,
and may configure it differently.

```
        Host
      /      \
Addon A      Addon B
      \      /
       Addon C
```

If a conflict occurs, Ember-CLI MUST throw an error and prompt the user to
resolve the conflict by overriding the configuration value for the conflicting
addon at a higher level. The Host's config always wins, so the final decision
can be made at the Host level.

## Propagating Values and Dynamic Configs

There are cases when Apps, Addons, or Engines may want to configure their
children based on their own config values. In order to do this, we introduce the
`getConfig` helper function.

The `getConfig` function can only be used in configuration files that are
located in the `/config/engines` and `/config/addons` directories. During the
build phase, the Host's main configuration files will be the first one's to get
evaluated, and then these folders will be walked. The function of `getConfig`
is to copy a value from one of the already evaluated main configuration files
into the current file.

The main usage of this function is for developers who want to fully encapsulate
their Addon or Engine, but provide users the ability to configure
sub-addons/engines that they have already included. Consider the following
example:

We have a an engine, `my-engine`, that includes an addon, `my-select`, which is
a theme-able community addon. The author of the select included two themes,
`red` and `blue`, and by default the selected theme is `red`. The author of the
engine also has two themes, and decides that `blue` should be the default, but
`red` could also be configured by the user.

The engine's config files look like this:

```js
// /my-engine/config/engine.js

export default {
  theme: 'blue'
};
```

```js
// /my-engine/config/addons/my-select.js

export default {
  theme: getConfig('my-engine.engine.theme')
};
```

And the select's config files look like this:

```js
// /my-select/config/addon.js

export default {
  theme: 'red'
}
```

Resulting in this config object by default:

```js
{
  "my-engine": {
    //...
  },
  "my-select": {
    theme: 'blue' // overwritten by `my-engine`
  }
}
```

In a consuming application, however, we could specify that we want to use the
red theme in our engine:

```js
// /my-app/config/engines/my-engine.js

export default {
  theme: 'red'
};
```

This would set the value in `my-engine`'s engine config file, which would then
be linked to `/config/addons/my-select.js` and propagated down to the Addon.

For cases when linking alone is not powerful enough, users can combine linking
with function values.

This Host's engine config:

```js
// /my-engine/config/engine.js

export default {
  theme: 'blue'
};
```

With this addon config:

```js
// /my-engine/config/addons/my-addon.js

export default {
  borderColor() {
    const theme = getConfig('my-engine.engine.theme');

    if (theme === 'blue') {
      return '#0000FF';
    } else if (theme == 'red') {
      return '#FF0000';
    }
  },

  backgroundColor() {
    const theme = getConfig('my-engine.engine.theme');

    if (theme === 'blue') {
      return '#9999FF';
    } else if (theme == 'red') {
      return '#FF9999';
    }
  }
};
```

Combined with this Addon's `addon` file:

```js
// /my-addon/config/addon.js

export default {
  borderColor: 'grey',
  backgroundColor: 'white'
}
```

Results in this config object:

```js
{
  "my-engine": {
    //...
  },
  "my-addon": {
    "engineTheme": "blue",
    "borderColor": "#0000FF",
    "backgroundColor": "#9999FF";
  }
}
```

## Building the Configuration

One of our design constraints is to calculate the configuration at build time.
This has two advantages:

1. Performance - Configurations are just POJOs that are easy and low cost to
access.

1. Simplicity - Configurations are NOT dynamic, making them much easier to
reason about.

The process for building configurations is as follows:

1. Begin at the root App as the first Host, and create a POJO (we'll call this
the Config).

2. Walk the `/config` directory and evaluate all javascript files. Place the
resulting objects on the Config under the name of the root application at a path
corresponding to that of the filesystem.

3. Walk `/config/addons`, evaluate each file, and store the results for the next
step.

4. For each addon, evaluate the files in `/config` and place the resulting
objects the Config under the name of the addon at a path corresponding to that
of the filesystem. When the `/config/addon.js` file is reached, first evaluate
it, then overwrite any values with those that were calculated for this addon in
the previous step.

5. For each of the addons sub-addons and so on, repeat steps 3-4. If an addon
is included more than once and has more than one `/config/addons` file, flatten
all instances of the result, with the highest parent winning. Throw an error
if values have already been added and conflict (this should only happen if two
separate addon trees exist, they have different configs, and they don't have
a parent config that overrides them both).

6. Returning back to the Host - walk `/config/engines`, evaluate each file, and
store the results for the next step.

7. For each engine, repeat steps 1-6 as with the engine as the new host, with
the only differences being that the output from step 6 should be merged into the
engine's `engine.js` file. If more than one mount point was specified in the
output of step 6, repeat steps 1-6 with engine as the host for each mount point
as well.

8. Store each config object somewhere that is accessible from the container of
the engine/app it belongs to.

These steps are meant as a general outline and don't mean to specify the
implementation - it may be much more performant to calculate the configs another
way.

# Drawbacks

* The differences between Engine and Addon configuration may be confusing to
users. In addition, conflicts between different versions of Addons within a
single Host may not be resolvable, and this could result in difficult to trace
bugs and unexpected behavior.

* Functions on configuration objects may result in users placing too much
business logic in configs.

# Alternatives

* If Addon's can be made more self-encapsulated, configurations may be able to
be specified for each instance of the addon in the dependency graph. The syntax
and methods for configuring Engines could be extended to Addons as well,
removing any differences that would be confusing and any possibility of an
unresolvable conflict.

* As mentioned in the Pull-based vs Push-based section above, this RFC uses a
Push-based model for propagating configurations. Using a pull-based model may
be another alternative.

# Unresolved questions

* How the configuration objects would be injected onto their respective
containers. Probably exported as modules and registered using an instance
initializer.

* When and where Ember-CLI should throw errors vs warnings regarding
configuration build issues such as conflicts.

* Ember-CLI should somehow make the final built configuration objects viewable
to developers, so they can debug their configs and figure out what the final
values are. What is the most ergonomic way to do this?
