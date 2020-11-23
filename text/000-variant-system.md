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

This system requires all source assets to be in `src/`.

A common directory, `src/common` is now used for storing shared assets/code. This directory contains code that will be applied to all variants, unless they specifically overwrite a certain asset.

Directories are then created for each varian1. For example:

```
src/
  |- common/
  |- 320-neo-leap/
  |- 320-classic-cfm/
  |- 321-neo-lr-leap/
  |- 319-classic-iae/
```

## Build-time behaviour

At build time, every variant ouput directory is initialized with the contents of the `common/` directory. Then, the files in the variant folder are copied, overwriting every file that already eexists in `common/`.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

TODO

# Drawbacks
[drawbacks]: #drawbacks

* Mental model of which files are there in the output can be complex
* Things can break if not thought about correctly

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

* Use a library-tree-like inheritance system
  * Every variant would "include" the common files through a dependency system
  * Needs everything to use an ESM-like import system
  * Advantages: requires more engineering
  * Disadvantages: can be seen as "overkill"
  

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
