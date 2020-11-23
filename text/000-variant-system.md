- Feature Name: System for dynamically building variants of MSFS plane addons
- Start Date: 2020-11-23
- RFC PR: [flybywiresim/rfcs#2](https://github.com/flybywiresim/rfcs/pull/2)
- A32NX Issue: N/A

# Summary
[summary]: #summary

TODO

# Motivation
[motivation]: #motivation

As the scope of FlyByWire projects eveolves, it becomes needed to develop multiple variants of one single aircraft. Examples of that include:

* A32NX - classic variants, A321, A319, etc.
* for all projects - allow beta-testing of features

In order to maintain the codebase to generate multiple variants of one addon, it is necessary to find a system in which certain parts of the code can be altered depending on which variant is currently being built.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

TODO

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

TODO

# Drawbacks
[drawbacks]: #drawbacks

TODO

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

* Use git branches
  * Advantages: system already exists, only CI script needs to be tweaked
  * Disadvantages: requires a lot of work to keep variant branches up to date with master, not what git branches are typically used for
  
* Use the native MSFS variant system
  * Does not cover code compilation and asset transformation
  * Requires MSFS SDK in CI (legal problems)
  * Based on a Windows executable

# Unresolved questions
[unresolved-questions]: #unresolved-questions

TODO

# Future possibilities
[future-possibilities]: #future-possibilities

TODO
