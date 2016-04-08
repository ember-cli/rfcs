- Start Date: 2016-04-08
- RFC PR: (leave this empty)
- ember-cli Issue: (leave this empty)

# Summary

The creation of the Ember Forge Foundation to function in the same vein as the [Apache Foundation](http://www.apache.org) and [Dojo Foundation](http://dojofoundation.org), for example. Some examples of their efforts include:

> innovation, by collaborative consensus based processes, an open, pragmatic software license and a desire to create high quality software that leads the way in its field
- [Apache Foundation](http://www.apache.org)

and

> provide just enough foundation to make great open-source projects succeed and be professional and trustworthy to their users, without bureaucracy or excessive process for projects and their contributors. - [Dojo Foundation](http://dojofoundation.org)


Would also provide for the maintenance and management of addons deemed popular or critical by/to the community, that if left unmaintained would be bad for the ecosystem.


# Motivation

While developing several applications and addons, and during conversations with other Ember developers, @juwara0 and @notmessenger have identified several parallel motivations for this effort which have been categorized below:

* Evaluation of available addons for use
    * Which one should I use when there are multiple similar ones?
    * Is it the one used the most by others?
    * Which ones have a proven track record of being maintained and supported?
    * How many maintainers are there?
    * Does it follow best practices?
    * Does it follow Semantic Versioning?
    * Which versions of Ember or Ember CLI are supported or required?
    * License and IP (CLA)

* Benefit of consistent experience across addons through standardization
    * Documentation of use
    * Documentation of API
    * Interfaces through adapters for compatibility and interoperability
    * Consistent approaches to similar concerns
    * Best coding practices
    * Composability (where applicable)
    * Accessibility support (where applicable)
    * Internationalization support (where applicable)

* Development of addons
    * Best practices (some overlap with best practices above)
    * Alignment of community efforts to:
        * reduce duplication of effort
        * build what the community desires
        * align with cross-cutting concerns of the ecosystem

* Identification of correct resources and approaches
    * Curated list of resources
    * Documentation
    * Cookbooks

* Community engagement
    * Prioritized list of needs within the Ember community



# Detailed design

* Evaluation of available addons for use
    * Which one should I use when there are multiple similar ones?
        * There could be an Ember Forge designation that indicates that community-supported standards are being followed and implemented within an addon.  An Ember Forge designation would have preferred usage over those without this designation.
        * The Ember Forge organization would work with similar addons to encourage the reduction of duplication of effort.
            * See the efforts of [ember-forge-ui](http://github.com/ember-forge/ember-forge-ui) which is the colloboration of two different UI component libraries in the spirit of the Ember Forge organization)
    * Is it the one used the most by others?
        * There could be an Ember Forge designation that indicates community endorsement of this addon.  An Ember Forge designation would have preferred usage over those without this designation.
        * Some of this data (implementation) already exists in Ember Observer and could remain there or additional colloboration could be implemented.
    * Which ones have a proven track record of being maintained and supported?
        * There could be an Ember Forge designation that indicates community endorsement of this addon.  An Ember Forge designation would have preferred usage over those without this designation.
        * Some of this data (implementation) already exists in Ember Observer and could remain there or additional colloboration could be implemented.
    * How many maintainers are there?
        * Part of being Ember Forge designated is the requirement of a minimum number of maintainers, spread across different organizations.  One such organization could be Ember Forge itself, of which individuals could be members of.  The goal of this is to remove a single point of failure in the maintenance of a repository.
        * Some of this data (implementation) already exists in Ember Observer and could remain there or additional colloboration could be implemented.
    * Does it follow best practices?
        * There could be an Ember Forge designation that indicates that community-supported standards are being followed and implemented within an addon.  An Ember Forge designation would have preferred usage over those without this designation.
    * Does it follow Semantic Versioning?
        * There could be an Ember Forge designation that indicates that community-supported standards are being followed and implemented within an addon.  An Ember Forge designation would have preferred usage over those without this designation.
    * Which versions of Ember or Ember CLI are supported or required?
        * https://github.com/mixonic/ember-community-versions could possibly assist with this.
        * http://emberup.co/introducing-embadge-io/ could possibly assist with this.
        * The enforcement/support of this via the usage of https://github.com/rwjblue/ember-cli-version-checker could be one of the criteria for being Ember Forge designated.
    * License and IP (CLA)
        * The use of CLAs would help keep clean IP within Ember Forge designated codebases.  Such assurances would assist with corporate adoption.
        * The Ember Forge organization could take on the management of this process so that individual addon maintainers do not have to.

* Benefit of consistent experience across addons through standardization
    * Documentation of use
        * The existence of usage documentation could be one of the criteria for being Ember Forge designated.
        * The Ember Forge organization could provide templates and best practices to follow for consistency amongst Ember Forge designated projects.
    * Documentation of API
        * The existence of API documentation (for components or other pieces that make sense) could be one of the criteria for being Ember Forge designated.
        * A best practice the Ember Forge foundation could evangelize is the usage of [ember-cli-jsdoc](https://github.com/softlayer/ember-cli-jsdoc) for this.
    * Interfaces through adapters for compatibility and interoperability
        * There could be Ember Forge provided adapters that codebases could leverage in order to ensure interoperability, even between repos that are not Ember Forge designated.
        * Examples include: translation and localization adapters, amongst others.
    * Consistent approaches to similar concerns
        * Ember Forge would achieve this through blog posts, adapters, mixins, addons, and community education.
    * Best coding practices
        * Ember Forge would achieve this through blog posts, adapters, linters, mixins, addons, community education, and the promotion of codebases utilizing and adhering to the same.
        * Some accurate content already exists in the guides and other locations.  This content could be part of a curated list that is referenced as well as content spread throughout that community that is Ember Forge compatible/approved (best practices, correct, etc) could be given visual indication as such.
    * Composability (where applicable)
    * Accessibility support (where applicable)
        * Part of the promotion of best practices.  Encourage and assist the development of the https://github.com/ember-a11y/ember-a11y and similar in a unified community approach, and then promote the adoption of this/these tool(s) into codebases.
    * Internationalization support (where applicable)
        * Part of the promotion of best practices.

* Development of addons
    * Best practices (some overlap with best practices above)
    * Alignment of community efforts to:
        * reduce duplication of effort
            * Identify duplicated efforts and encourage coordination and consolidation
        * build what the community desires
            * Determine such desires and prioritize them
            * Engage the community to actively develop against this list
        * align with cross-cutting concerns of the ecosystem
            * Help coordinate activity and architecture of addons based on known needs and efforts in other projects and efforts within the ecosystem
            * A review board provides feedback on Semantic Versioning breaking changes as to whether it aligns with community goals and efforts.  An addon can choose not to follow any such guidance, if it is not in agreement with the proposal, but there will be a public history of such discussion.

* Identification of correct resources and approaches
    * Curated list of resources
        * Some accurate content already exists in the guides and other locations.  This content could be part of a curated list that is referenced as well as content spread throughout that community that is Ember Forge compatible/approved (best practices, correct, etc) could be given visual indication as such.
    * Documentation
        * Some accurate content already exists in the guides and other locations.  This content could be part of a curated list that is referenced as well as content spread throughout that community that is Ember Forge compatible/approved (best practices, correct, etc) could be given visual indication as such.
    * Cookbooks
        * Some accurate content already exists in the guides and other locations.  This content could be part of a curated list that is referenced as well as content spread throughout that community that is Ember Forge compatible/approved (best practices, correct, etc) could be given visual indication as such.

* Community engagement
    * Prioritized list of needs within the Ember community
        * Top N wish list?
        * Some examples:
            * documentation for [ember-weakmap](https://github.com/thoov/ember-weakmap) that Stefan spoke of in the "Building Stateful UI" in the EmberConf 2016 training session
            * documentation of specific areas
            * library of common UI components (see the efforts of [ember-forge-ui](http://github.com/ember-forge/ember-forge-ui) which is the colloboration of two different UI component libraries in the spirit of the Ember Forge organization)
        * The reason for this is to concentrate effort and resources where it will have the most (desired) impact


Being Ember Forge designated means:
* Serves as an example of accepted best practices
    * In architecture and philosphy
        * For example, [10 Considerations When Developing Ember CLI Addons](http://notmessenger.com/considerations-when-developing-ember-cli-addons/)
    * In coding
    * In testing
        * For example
            * [Testing best practices](https://github.com/softlayer/ember-style-guide/blob/master/ember-testing.md)
            * [No usage of global libraries in components](https://github.com/softlayer/sl-ember-test-helpers#global-libraries)
    * In release management
    * In repo management
        * Roadmaps
* Follows a consistent approach to solving problems and applying patterns
* The removal of a single point of failure from an administration standpoint, so that the codebase is known to dependable



# Drawbacks

Why should we *not* do this?

* Community may not believe there is a need for it.
* May be too soon in the Ember community lifespan for such an effort.
* May be viewed as too burdensome, heavy-handed, or misguided.

# Alternatives

What other designs have been considered? What is the impact of not doing this?

* Not providing any of this guidance to the community.

# Unresolved questions

* Have not researched when other similar organizations came into existence in relation to the ecosystem they were supporting.
* The path similar organizations followed on their way to existence has not been researched.
* What are the criteria for becoming Ember Forge designated?  Possible data points:
    * Ember Observer score
    * number of downloads/installs
    * number of issues/length of time before an issue is resolved
    * popularity (number of GitHub stars?)
    * would need to adhere to Ember Forge best practices
* Is there a difference between being Ember Forge designated, to imply quality of solution, versus Ember Forge managed, where a repo has been deemed critical to the ecosystem and the Ember Forge foundation has the ability to manage it, reducing the single point of failure?  In the latter, do any of the quality metrics matter?
* The minimum number of maintainers attached to a Ember Forge managed (see above bullet point) project and how many distinct organizations they need to represent.  All individuals is fine, several from the same organization may not be.
* How are these maintainers (see above bullet point) vetted for inclusion in the group of maintainers?
* Periodic review of Ember Forge designated reposorities for continued compliance.
