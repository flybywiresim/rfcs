- Feature Name: performance_parameterization
- Start Date: 2020 12 26
- RFC PR: [flybywiresim/rfcs#0000](https://github.com/flybywiresim/rfcs/pull/0000)
- A32NX Issue: [flybywiresim/a32nx#0000](https://github.com/flybywiresim/a32nx/issues/0000)

# Summary
[summary]: #summary

Link variant-specific parameters of MCDU/FMGS/AP to be read form externa config file.

# Motivation
[motivation]: #motivation

Allow other developers to quickly utilize the FBW A320, while tuning variant specific parameters for the application.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

If the change were made, the following parameters (which would vary from A320neo to A321neo to A319neo etc.) would be contained in an external .cfg file:
- Optimum cruise altitude matrix for various gross weights and CI
- Various weight and fuel capacity limitations
- V1/Vr/V2 for various conditions
- Climb/cruise/descent fuel/time/distance prediction figures
- Autopilot tuning parameters that may vary from one variant to another
- etc.

These parameters would be read by the systems created as part of the FBW A320 cockpit on loading the initial aircraft. 

Doing so would allow other developers to use the exact same cockpit, except provide their own .cfg file. This cfg file would then be loaded on initial boot-up of the aircraft, so an A321 would be using the FBW A320 cockpit with tuned performance figures to ensure compatibility.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

N/A; I don't understand the technical details enough to propose anything (I'm just a user).

# Drawbacks
[drawbacks]: #drawbacks

May not be worth the effort if FBW plans on making their own A320 family package.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best in the space of possible designs?
Simply, it alleviates workload from other developers, while ensuring that the end-user sees normal expected behavior. At the end of the day, the onus is still on the 3rd (4th?) party developer to ensure that their selected performance figures are correct, and not FBW.
- What other designs have been considered and what is the rationale for not choosing them?
Unknown
- What is the impact of not doing this?
Users would possibly see unexpected behavior from 3rd (4th?) parties using the FBW A320 cockpit in their designs, such as unrealistic performance figures or simply see errors 

# Unresolved questions
[unresolved-questions]: #unresolved-questions

Is it even possible to load a 3rd party's cockpit while providing your own config files? Would the cockpit be able to read these parameters?

# Future possibilities
[future-possibilities]: #future-possibilities

N/A
