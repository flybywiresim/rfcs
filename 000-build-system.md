- Feature Name: Build system
- Start Date: 2021-02-13
- RFC PR: [flybywiresim/rfcs#11](https://github.com/flybywiresim/rfcs/pull/11)

# Summary
[summary]: #summary

Modular and configurable system for building MSFS aircraft packages.

# Motivation
[motivation]: #motivation
After months of developing the A32NX using in-repo, bespoke build tools, a few obvious drawbacks have appeared as both the scope of development and number of projects have increased.

* The build system is not clearly defined in its geometry or operation model
  * There is no clear definition of "target", "task", "step" or "project".
    * This limits the amount of correctness that can be enforced in task definitions, order, input or output 
    * This prevents performing generic step-running operations such as step skipping and output caching
    * This prevents the easy implementation of differential build systems that produce different packages depending on environment or configuration
* The build system is contained entirely within the A32NX repository
  * This limits its usage to the A32NX project
  * This prevents the centralization of code and the existence of a single source of truth if multiple projects use the system
  * Additionally (but not as a consequence of that), many values are hard-coded to generate A32NX packages

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

The proposed build system is a tool that, simply put, transforms a set of source files into a built and ready-to-use MSFS aircraft package.

To operate, it requires:

- The aforementioned source files
- A project configuration, describing the project it is building
- A build schema, describing different build tasks
- Build configs, describing configurations that use different tasks based on environment

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Drawbacks
[drawbacks]: #drawbacks

* Engineering time is significant

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

* No alternatives

# Unresolved questions
[unresolved-questions]: #unresolved-questions

Any unresolved questions you may have about this feature and its possible implementation.

# Future possibilities
[future-possibilities]: #future-possibilities

How this feature or change could be expanded on in future PR's and RFC's.
