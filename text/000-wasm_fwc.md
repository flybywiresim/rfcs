- Feature Name: `fwc_modules`
- Start Date: 2020-12-21
- RFC PR: [flybywiresim/rfcs#0000](https://github.com/flybywiresim/rfcs/pull/0000)
- A32NX Issue: [flybywiresim/a32nx#0000](https://github.com/flybywiresim/a32nx/issues/0000)

# Summary
[summary]: #summary

This RFC proposes a modular structure for the Flight Warning Computer (FWC) that will improve maintainability code organization, allow more realistic features in future and approximates the real system layout.
There is also a good chance this structure will improve performance.

# Motivation
[motivation]: #motivation

Currently the flight warning computer's tasks are fully implemented in Javascript and split up into a logical part (as `A32NX_FWC` in a core folder) and the message and display part (in the `UpperECAM` Javascript file).

The task distribution is as follows:
- `A32NX_FWC`: Calculate the current flight phase and some timers
- `UpperECAM`: Determine the list of active/inactive warnings, inhibit and recall behaviour, sounds

With the current structure, features like multiple independent FWCs and display switching are tricky to get right. The new structure enables these features and follows the design of the real FWC.

Finally, by moving the final string assembly into the FWC itself we're closely approximating the real system layout, where the final FWC output is passed to the display management computers (DMCs) that don't care about the warning logic at all and are just concerned with rendering the FWC's output.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

The new Flight Warning Computer is organized as follows:
- WASM FWC: We have a full WASM implementation of the FWC, that consists of signal acquisition, warning and memo logic, and the monitor.
- UpperECAM: We have a piece of Javascript code that can take the final FWC output and render it in HTML with markup. This lives in the upper ECAM for now, but may eventually become the DMC logic.

## WASM FWC

The WASM is fully implemented in Web Assembly. This should improve performance.

### Signal Acquisition

Due to the nature of the FWC, it requires signals from most (if not all) aircraft systems. At first this primarily means reading dozens to hundreds of SimVars, but allows for progressive improvements:
In the mid-term we may be able to directly read signals from other systems implementing in WASM without needing to write all of them to SimVars. As an example, we may read detailed signals from the hydraulic or electrical system that are not required in any other place (as they may be too detailed to upper on the lower ECAM system page).
In the long-term we will probably also replace the "true" SimVars from the simulator with a synthetic sensor system that supports multiple soures. As an example, currently we're using the RADIO ALTITUDE simvar. In future we may have a sensor system that reads RADIO ALTITUDE, introduces some bias or aircraft orientation and outputs sigals for radio altimeter 1 and radio altimeter 2. Failures may invalidate or falsify the signal from these sources.

By enapsulating the signal acquistion into a single part of the system, we can transparently replace these sources over time without having to rewrite the warning logic itself.
To gain performance improvements we could also centrally control the refresh rate of this module alone, for example in order to let users with weaker systems have the FWC refresh all the SimVars only once a second rather than multiple times.
For testing purposes we can also replace the signal acquistion with mock signals, allowing us to easily test the other parts of the systems without having to recreate the flight situation. As an example, we may want to simulate a complex, but inhibited failure during takeoff. By mocking an appropriate aircraft speed, wheel status and fault signals we can assert whether an ECAM message is shown or correctly inhibited.

### Warning and Memo Logic



### Warning and Memo Definitions

Each warning or memo is implemented in it's own class. This helps us isolate code for unrelated warnings from each other and ensures easy maintainability.


### Monitor

The monitor is the module responsible for prioritizing, grouping and inhiting warnings and memos. Where the previous modules independently determine flight phases and the logic status of single warnings or memos, this monitor reconciles these together while respecting activation delays and the clear, emergency cancel and recall buttons.


# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

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

## Replacing SimVars with

## Sensor system and failures
A crucial role of FWC is to help pilots detect and react to a variety or problematic aircraft conditions. In order for pilots to enjoy it to it's full potential, problematic aircraft conditions should actually be possible, even in fully normal and compliant operation.

This allows the system to develop it's own quirks and ultimately 


