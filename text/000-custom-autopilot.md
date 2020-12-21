- Feature Name: custom-autopilot
- Start Date: 2020-12-21
- RFC PR: [flybywiresim/rfcs#0000](https://github.com/flybywiresim/rfcs/pull/0000)
- A32NX Issue: [flybywiresim/a32nx#0000](https://github.com/flybywiresim/a32nx/issues/0000)

# Summary
[summary]: #summary

This feature is about the introduction of a custom autopilot to a32nx.

# Motivation
[motivation]: #motivation

This feature is the foundation to implement all possible modes on the A320, especially the managed modes. The second reason is to get independent of the default AP.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

I need the logic and interface to the FCU...I‘m currently „only“ working on the base laws. There is some switching in but only for testing.
Ah and it’s good to have LNAV and VNAV also in focus.

As you’ve seen the H/PATH law is already there and I did some tests. It works, still there is a need that the path being provided and the capability of the aircraft needs to be aligned. Otherwise it’s not smooth. 

Regarding the VNAV, according to the AMM the FMGC just delivers an altitude and vertical speed to the AP. So that’s simple from AP perspective.

Fun fact: the ALT HOLD is not dependent on barometric pressure setting.

It would be good to have the events that can happen from the FCU and also have signals for all the displays. All the logic should go into the wasm (so e.g. turning a knob only raises events). That it's possible to fix flaws like the heading change (not exact values) or speed/mach (mach controls hidden speed and no every tick changes the number).

The logic for the switching of base laws (example: V/S, ALT*, ALT) is not within them. The OP CLB/DES is one base law, but it does not tell when to switch to ALT* or ALT. In that case there also needs to be a logic that then commands the A/THR to IDLE or CLB thrust.

What is needed is kind of a state machine, that take at first the discussed FCU inputs and produces output signals for FCU and FMA. Secondly it takes into account all the mode transitions according to documentation. For the start not all transitions are needed.

A good start could be to have special events from the FCU (maybe in parallel to the existing).

The base laws will be provided in a format that take a certain input and produce a certain output (in that case at least Pitch and Bank Angle commands for AP and FD, later also a ß for autoland). Those can be directly feeded into a special interface of the custom FBW.

It is suggested that the current custom FBW interface code (not the FBW model itself) is ported over to Rust. This will allow integration with the custom A/THR but one can also take advantage of the simpler SimConnect interface in Rust.

Important: the current available features of the custom FBW interface code like thrust lever mapping or flight data recorder need to be kept. The port to Rust should not be visible to any user.

Ruled out for the start (either because it's very complicated or dependent on other stuff):

- Autoland (display amber message "DISCONNECT AP FOR LDG" in FMA cols 2+3 when AP/FD remains enganged below Minimum - 50 ft or 400 ft AGL when no minimum has been entered)

Restricted function or performance expected:

- NAV
- CLB / DES modes (OP CLB / DES or V/S to be used for the start)

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

Needs to be filled on the go...

# Drawbacks
[drawbacks]: #drawbacks

A lot of tuning is necessary and there are many situations out there that maybe cannot be tested.

Risk mitigation is to add the right and enough parameters to the flight data recorder so an analysis of situations can take place.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?


# Unresolved questions
[unresolved-questions]: #unresolved-questions

Any unresolved questions you may have about this feature and its possible implementation.

# Future possibilities
[future-possibilities]: #future-possibilities

How this feature or change could be expanded on in future PR's and RFC's.
