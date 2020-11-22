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

**Current known issues with flight plan system**:
- USER and D0 waypoints should not be present.
- Extremely wide/shallow turns are made when following a flight plan with a sharp (>90 degree) turn at a non-overfly waypoint. It can often take several nautical miles for the plane to re-intercept the track to the following waypoint.
- (T-P) waypoint should be present when intercepting a direct-to waypoint.
- VERT REV page: can't set/change/clear speed constraints.
- VERT REV page: can't clear altitude constraints.
- Can't delete or perform a vertical/lateral revision on a STAR waypoint.
- Can't go DIR TO some STAR waypoints (need to investigate when/under what conditions this happens).
- VERT REV page: can't set "above" altitude constraints (+). Instead, the MCDU changes it to a "below" (-) constraint once entered.
- VERT REV page: can't properly set "below" altitude constraints: Inserting with "-<number>" does nothing. See point directly above for related behavior.
- VERT REV page: can't properly set "at" altitude constraint (no + or -).
- Transition/TRANS and (DECEL) waypoints often have inaccurate altitude constraints, such as 100 or 1000 feet for example, when the previous and next waypoints' altitude constraints are correct.
- Trying to fly an ILS approach without a transition and/or STAR may cause the plane to instead fly directly to the airport upon reaching the first approach waypoint
- Direct-to any STAR waypoint causes undefined behavior, including:
    - Deleting previous waypoints from the navigation display, except departure airport.
    - After reaching/passing the direct-to waypoint, the departure airport is deleted and replaced with a USER waypoint.
    - After reaching/passing the direct-to waypoint, some other remaining waypoints in the STAR may be skipped, and the flight plan may go direct-to a different waypoint in the STAR which is not the upcoming one.
    - If a second DIR TO is performed on the STAR to correct this error, the plane may fly the flight plan in reverse after reaching that DIR-TO waypoint. Trying to go direct-to another STAR waypoint to fix this may cause a complete deviation from the flight plan and more undefined behavior.
- DIR TO input field ("[   ]") next to LSK 1 holds the waypoint name of the previous waypoint that a DIR TO operation was performed on, when this field should be cleared once the direct-to has been completed.
- Executing a DIR TO (by pressing "INSERT*"/"TMPY INSERT*") should engage managed heading (nav) mode if currently in selected mode - this behavior is not implemented it seems.
- Executing a DIR TO on a waypoint will cause the green "flight plan" line on the ND (navigation display) from the current position to the waypoint to be cut off just before the waypoint, instead of joining up with the remaining flight plan line in a smooth, continuous fashion.
- Updating the altitude (and most likely speed, when implemented) constraints on a waypoint will not update their values on the ND (navigation display) when the CSTR EFIS filter is currently active - you must turn CSTR off then on again for the ND values to update, which should not be necessary in real life.
- VNAV: Waypoint altitude constraints are not obeyed in managed mode, at least at first glance. Needs more investigation.

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
