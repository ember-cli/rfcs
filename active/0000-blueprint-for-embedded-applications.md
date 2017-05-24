- Start Date: 2015-08-19
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

Ember-CLI to support embedded application blueprint.
Generating an adjusted base application blueprint with the intent for the application to exist within another website.
Providing an option for using Ember outside of a full website build while still leveraging ember-cli.
Giving Ember a larger field to play on.

# Motivation

Let me tell you the story of Acme Financial. A large enterprise wall street firm. Acme Financial has a website that does all your banking an financial needs. For their investment accounts, they have available a stock screener. The existing is fully functional, but lacks the latest style and user interaction, making it look and feel out dated. To keep customers happy, Acme Financial decides to rebuild a screener, using a modern interface. The old screener had a large list of filters, each with native form components for selecting options, and a tiny server-side call that would fire on change of any of these, that would simply return the total number of results without showing them. Then the user would click "Display Results" and a new page from the server would render, displaying their results. They would have pagination, but they would have to go back to the previous page to change their filters.

Acme Financial wants to make the new screener completely live. To have the filters and results on the same page, auto-updating the results table on each filter change. With a sleek look and feel that with draw in new customers and keep existing ones. Acme does need these tool to work in their existing site however, which is pretty massive already given their enterprise level size. The dev team talks and decides the best course of action is build a small and secure REST API to expose their backend screener calls and build a client-side application. And application that will render in place on the existing site. Great plan! Now what tools are available to build this **ambitious application**?

The Acme dev team sets out to look for the best tool for the job.

Angular looks good. it's a run-time environment framework that we template and define controllers to wrap up all the interactivity in a nice package, and create directives for the more customized components. We'll have to figure out what the best way to structure it with how our data is designed, but that's ok. Should we use 1.x or starting developing on 2.0? Lot to think about here...

React looks good. It's designed for exactly what we are trying to build, an enhanced user experience. It has the capabilities to do what we want., but it looks like we'll have to handle the ajax calls ourselves and manual set our state. And we can use it right in our existing site. This is a young framework, and it's not at 1.0 yet. It is really stable enough to use in production?

Ember looks good. It's got all the capabilities of the other two, and has a data persistence layer to boot. Ember-cli is great, the convention over configuration and the standard application structure will really speed up our development. But wait... the cli want's me to build a whole site. How do I create just an embedded application? (20 minutes of googling later) Oh I see, but these examples are old and don't use ember-cli. I remember reading too that ember-cli is going to become a first-class citizen of ember, is building an ember app today with the cli a good idea?

After careful evaluation, the team determines that Ember is the most mature of the three frameworks and is the best choose for a long-term solution for their *ambitious* new screener *if* they were building a stand-alone site for it. However, that isn't the case here. The team knew that a requirement for their new screener is that it needed to exist in their current site. The winner, eventually, went to Angular.

I have been a part of this story, and have heard it many times over from other developers. As amazing as ember-cli is for building a full site in Ember, the problem is that it **only** builds a full site in Ember. As we all know, Ember has the capability to do exactly what Acme Financial's screener needs, but is overlooked or ruled against due to limitations of the ember-cli blueprint for a monolithic site. Let us not lock ourselves into this. Let us along our build tool to be more variable. Let us have blueprints for a variety of use cases for Ember, and give back the option for embedded application to our developers!

# Detailed design

First let me speak towards the scenario of an embedded ember application. A page will be rendered from the server, from whatever framework the client is using. Once the page is rendered and `Ember.Application.create();` is called, Ember will render into the `rootElement`. The `Router` will most likely only have a single top-level route, and all other routes are embedded into that. `Router.location` will be set to `none`. The page's main navigation, header and footer, side bar are all rendered from the server just like every other page, but the embedded application is all Ember.

With that said, how do we allow Ember-cli to have builds for this type of setup?

My two solution ideas are:

- a different application blueprint for embedded apps `ember new --embedded` for example will create a different blueprint

or

- `ember build --embedded` will build the application slightly differently to better allow for the given use case

A different initial blueprint would not be much different from the current one, just geared towards an embedded application. it would not include `app/index.html` for example, and the `storeConfigInMeta` option for `EmberApp` would always be `true`. The build would also not include `robots.txt` and the such. A dev preview build *will* however include a `dist/index.html` with the normal livereload options, etc, for the purpose of development (of course), but a prod build will only including the compiled vendor.js and my-app js (and style files if necessary).

Normal required dependencies would also be optional for the `build --prod` as while they will be needed for development and testing, they may not be needed for production (e.g., jquery may already be defined globally, so no need to included it in the vendor bundle).

The application bundle should also have the inclusion `require("my-app/app")["default"].create({});` be optional. A developer may decide to only serve up `my-app.js` and `vendor.js` from `dist` as part of just the page that contains the embedded application, or they may bundle them with the rest of their js as part of their standard asset build (e.g., ember dist js files will be concatenated and minified along with the rest of the javascript assets for the site). In that case `require("my-app")["default"]` should be exposed globally so a developer can call `.create({})` at their whim.

The `public` folder will not be included in the embedded blueprint. It should not be needed under the context of an embedded application. If we go the `ember build --embedded` route, it would not be copied over into `dist`.

Admittedly, I'm not nearly familiar enough with the ember-cli code base to give a more detailed approach to accomplishing this goal, or give examples of initial implementation, however I don't believe this to be an overly challenging thing and I doubt major refactor would be needed.

The remaining questions of "what should be included/added/removed" to/from the current blueprint I'll leave up to discussion.

# Drawbacks

The major drawback I see to this addition is to the current add-on structure. Ember-cli add-ons are currently designed to work with the existing blueprint. While many should continue to work on a new blueprint, many will not. Mostly ones that include additions for `{{content-for 'head'}}`, etc. Ember-cli add-ons may have to be aware of their compatibility.

# Alternatives

None so far

# Unresolved questions

What does the community think of this?

Is this one of the directions that Ember should be taking?

What if I wanted to build two Embedded apps in a single build? For example, Acme Financial had so much success with their stock screener, they decide to copy it over but for Mutual Funds. 80% of the functionality exists in the old one, only real differences are going to be the filter fields and some of the templates. Should we just build those into the existing application and have to option of which screener view? Could on two separate pages we could call the same `Ember.Application.Create({})` and then force it to display the desired page?