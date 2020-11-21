# FlyByWire RFCs

Most changes to FlyByWire aircraft and utilities can be implemented and reviewed through the normal Github pull request workflow.

However, some major changes may require extra thought and consideration, especially with the input of other contributors and the community.

The "RFC" (request for comments) process is intended to provide a consistent and controlled path for major new features or overhauls to the A32NX, A380, or other non-aircraft utilities published by FlyByWire Simulations.


## When you need to follow this process
---

You need to follow this process if you intend to make "substantial" changes to any FlyByWire aircraft or utilities. The definition of a substantial change may vary and evolve, but in general it is something that changes the code structure, deployment/build process, and/or interface. 

*This section needs more elaboration and specific conditions for an RFC.*

## Before creating an RFC
---

Please spend time to make a quality proposal, including as many details as possible. We urge you to speak with other contributors on our [official Discord server](https://discord.gg/flybywire) to pursue feedback before opening an RFC.

## Process for opening an RFC
---

- Fork the [FlyByWire RFC repository](https://github.com/flybywiresim/rfcs).
- Copy `000-template.md` to `text/000-my-rfc.md` (where "my-rfc" is descriptive). Leave the number as 000 - do not assign an RFC number yet as we will do so once the PR is merged.
- Fill out the RFC, taking care to go into detail about the design, possible impact, drawbacks, and alternatives.
- Submit a pull request, which will be commented on by the community and receive feedback
- The FlyByWire development team will discuss the RFC, taking feedback into account
- At some point, a development team member will propose a final vote on the RFC - whether to incorporate it or reject it. Either decision must reach a two-thirds majority vote with the development team, otherwise it will be postponed and await further feedback and changes

### Attribution
---
The format of this RFC process is based off the Rust language RFC, found [here](https://github.com/rust-lang/rfcs)