- Feature Name: **flightplan-rewrite**
- Start Date: **2020-11-21**
- RFC PR: [flybywiresim/rfcs#1](https://github.com/flybywiresim/rfcs/pull/1)
- A32NX Issue: N/A

# Summary
[summary]: #summary

This is an RFC to rewrite the default FlightPlanManager class (and related classes), and replace/rewrite all existing fs9gps bindings.

# Motivation
[motivation]: #motivation

The current FlightPlanManager, and associated Coherent and fs9gps calls, is severely lacking in functionality and contains numerous bugs. A rewrite will allow full support for lateral and vertical revisions of departure/arrival procedures, implement holds, implement secondary flight plans, and fix existing waypoint and routing issues. It may also improve performance.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

A new set of classes, prefaced by `FBW`, will be introduced, replacing those which are currently present in `\asobo-vcockpits-instruments\html_ui\Pages\VCockpit\Instruments\Shared\FlightElements\`. 

The primary driver class will be `FBWFlightPlanManager`, which replaces the existing `FlightPlanManager` class. Rather than interface with Coherent calls and fs9gps directly, it will instead manage and provide interaction methods for a **new** class called `FBWFlightPlan`, which will encompass a new data structure for flight plans which allow for the new functionality mentioned in the previous section of this RFC. A second new class will also be introduced, called `FBWAsoboDriver`, which will be updated by `FBWFlightPlanManager` and provide a simplified flight plan back to the sim via Coherent and fs9gps calls, so that the plane is still able to function with ATC and the default flight plan creation menu.

In order for the plane's managed autopilot systems to continue functioning, even with this new system, two key changes must be made. Target altitude simvar calls will be changed to pull constraint data from `FBWFlightPlan` rather than the sim itself. Lateral navigation simvar calls (such as `GPS COURSE TO STEER`) will be overridden to pull course data from `FBWFlightPlan` instead.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

TODO

# Drawbacks
[drawbacks]: #drawbacks

Would require extensive work spanning multiple weeks and could possibly lead to some minor inconsistencies with the new flight plan system (directly reflected in the Navigation Display and MCDU), and the in-game ATC and VFR map.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

TODO

# Unresolved questions
[unresolved-questions]: #unresolved-questions

To be debated: whether this rewrite should continue using JavaScript or make use of TypeScript.

# Future possibilities
[future-possibilities]: #future-possibilities

Can be expanded upon in future RFC's or PR's to add support for holds and secondary flight plans.
