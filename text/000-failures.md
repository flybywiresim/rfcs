# Summary

Whether it's the thrill of flying without knowing when failures will hit you, or the ability to test your skills in a failure scenario you set up; equipment failures add an exciting level of simulation on top of what we've built thus far.

This document discusses some of the functionality we could provide and provides a software design for its foundation.

# Requirements

_Equipment_ is some piece of the aircraft, whether hard- or software, which is necessary for its operation.

A _Failure_ is the abnormal functioning of a piece of Equipment.

A _Trigger_ is a cause or means through which a failure comes into existence.

Some example triggers that could be implemented are listed below:

- **Immediate**: immediately executes.
- **Timed**: executes after a certain duration.
- **Chance**: executes based on a certain chance per duration.
- **Real-world failure rate**: executes based on per component real-world or MTBF statistics.
- **Stress/misuse**: executes when exceeding the limits of the component.
- **Velocity**: executes when exceeding a certain velocity.
- **Altitude**: executes when exceeding a certain altitude.

A failure can be repaired. The realism level could be a configuration item. When realistic one can only repair failures on the ground. Otherwise, repair can be done at any stage of the flight.

@BehEh provided some excellent suggestions concerning failures:

> I've been thinking about a failure setup that is a bit more gamified and orchestrated by a director - similar to games like Left 4 Dead and Rimworld, where a director is in charge orchestrating the failures without the user having to select specific systems. For example:
>
> - Failing a random number systems when starting the aircraft but making sure to stay compliant to the MEL ("Let's see how challenging our flight will be today", without causing an emergency or the flight to end immediately alone).
> - Failing systems unpredictably one by one to eventually reach a pre-coded scenario with a very specific failure configuration (things that would be very, very rare in reality because of redundancy, but are interesting - e.g. multi ADR failure).
> - Having the omniscient director take into account factors like distance to nearest airport, equi-time points or airport equipment to selectively fail important systems for that flight phase to make a challenging, but salvageable situation (maybe the user can select the difficulty).
>
> The idea here is that it's more about indicating to the aircraft "I'd like an unexpected/tough/basically-impossible situation" without having to code things one by one. I'm thinking this system would be an overlay on top of the failure system itself.

Thus, a _Director_ would essentially provide a predefined or generated script (also called a _Scenario_), which triggers failures. We'll not get ahead of ourselves, but this is interesting stuff to add indeed!

The failures, triggers, director and scenarios are configured through the EFB. Note that this means that making the EFB a piece of failable equipment therefore might not be the best idea. 😉

If at any point in the future we store data on the aircraft which we can reload on the next flight, then failures should also be stored. Similarly, any stress-based information should also be stored.

# Design

Failure configuration is done using the EFB. This doesn't necessarily mean the responsibility of triggering failures (lets call it orchestrator for now) should also lie there, although it is the most likely candidate.

One of the main problems we face is having our equipment declared in multiple languages and environments. Systems are in a Rust built WASM, other systems and computers are in JS running within various displays, autopilot and FADEC are C++ built WASM. One would preferably not know from the perspective of the orchestrator which piece of software executes the failure.

The communication protocol is also highly limited, with some parts of the whole system only being able to communicate floating point numbers.

## Communication

Consider the orchestrator to be the publisher, and the various environments (Rust WASM, C++ WASM, JS per screen) consumers.

### Most triggers

Each failure has its own numeric identifier (_F<sub>i</sub>_), which is known both by the publisher and exactly one of the consumers. The publisher has a queue of failures. For each failure in the queue the publisher writes _F<sub>i</sub>_ to a SimVar _SV<sub>fail</sub>_. Consumers read _SV<sub>fail</sub>_ and determine if they are responsible for handling _F<sub>i</sub>_. If so, they execute the failure within their environment and overwrite _SV<sub>fail</sub>_ with _0_. If not, they leave _SV<sub>fail</sub>_ as is. When the queue contains another identifier, the publisher keeps observing _SV<sub>fail</sub>_, waiting for it to become _0_ and then writes the next identifier. It is important to note that SimVar writes are asynchronous, and thus a SimVar's written value might not be visible on the next frame. Therefore, the publisher should take care not to write values too quickly.

Failure repair involves another SimVar _SV<sub>repair</sub>_ through which _F<sub>i</sub>_ may be communicated to repair the failure. Here I specifically use _F<sub>i</sub>_ instead of _0_ (fail) and _1_ (repair), due to the asynchronous nature mentioned above. If instead of having one SimVar for failures and one SimVar for repairs we had one SimVar for _F<sub>i</sub>_ and another for a boolean indicating failure or repair, then there would be no means through which we would with certainty determine that the pair _(F<sub>i</sub>, bool)_ reflects the desired command.

This communication design is simple and should work well enough for most cases. The real-world failure rate and stress triggers are exceptions to this.

### Real-world failure rate trigger

For real-world failure rate we probably ought to accept that the failure rates are defined within the orchestrator, and thus will be re-defined per aircraft, even though they might contain the exact same equipment.

### Stress/misuse trigger

In this section, the orchestrator becomes the consumer, and various environments the producers.

Stress/misuse triggers are special as only the equipment itself can detect the abuse. Thus, we cannot give this responsibility to the orchestrator. However, the orchestrator does need to be aware of the failure as otherwise there is no way for it to indicate the failed equipment to the user and thus the user wouldn't be able to repair the failure.

Due to the asynchronous problem described above and the fact that this case has multiple publishers and a single consumer, simply writing another SimVar with _F<sub>i</sub>_ leads to problems. Thus we'll probably need to introduce two additional SimVars: _SV<sub>lock</sub>_ to which an environment writes its own unique identifier, and _SV<sub>misuse</sub>_ to which the environment writes _F<sub>i</sub>_ after the lock has been acquired, which can be verified by checking if _SV<sub>lock</sub>_ contains the unique identifier of the environment.

The consumer will read _SV<sub>misuse</sub>_ and overwrite it with _0_ after reading. The publisher itself will then either remove the lock written to _SV<sub>lock</sub>_ or write another _F<sub>i</sub>_ into _SV<sub>misuse</sub>_. The responsibility of removing the lock remains with the publisher, as giving this responsibility to the consumer could lead to a situation where the publisher reads _SV<sub>lock</sub>_ and _SV<sub>misuse</sub>_, with the following result: _(ID, 0)_, and then incorrectly assumes it can write the next _F<sub>i</sub>_, while in fact _SV<sub>lock</sub>_ has become _0_ right after the read.

### Recovering from failures

A case not yet covered is that of failure recovery itself, achieved by performing various actions on the piece(s) of equipment. For example, turning an engine generator OFF and then ON again might recover its function. Failure recovery follows the same communication pattern as described in the [stress/misuse trigger](#stressmisuse-trigger) section above, with the additional of a _SV<sub>recovery</sub>_ SimVar.

## Failure ranges

From a software point of view, it does not matter which numeric identifier _F<sub>i</sub>_ is used for which failure. For ease of understanding however, using ranges per area is helpful (for example ELEC, HYD). For example:

_ELEC = F<sub>1</sub>,⋯,F<sub>999</sub>_

_HYD = F<sub>1000</sub>,⋯, F<sub>1999</sub>_

Should an area ever exceed the range allotted, another range can be given to it.

## Systems.wasm

The Rust systems code has a lot of functionality which is generic. A failure code on the A320 is unlikely to be the same as a failure code on the A380. Thus, care should be taken to translate the failure code within the `a320_systems_wasm` crate to a failure message that is generic.

Within systems.wasm, a piece of equipment is a (part of a) `SimulationElement`.

On first thought, one could consider using the `SimulationElement.read` and `SimulationElement.write` functions for reading and writing failure information within each `SimulationElement`. However, this leads to various small problems:

- Unlike simulation variables changes, failures happen rarely, and thus continuously reading failure information is inefficient.
- Failures due to stress/misuse should not be written towards the sim every frame. They should only be written once. By including this in the `SimulationElement.write`, we would give the responsibility for not repeating writes to each individual element.

As such, a more elaborate approach is probably warranted. A potentially better solution is introducing a `Failure` type of zero or more instances are stored within each element.

The `Failure` type would contain the following capabilities:

- A function to query whether the failure is active.
- A function through which the element can indicate the failure became active (due to stress/misuse).
- A function through which the element can indicate the failure is recovered.

To facilitate the first point, another function would be added to `SimulationElement` through which a visitor would write failures coming from the simulator. This visitor would only be called when a failure is received, and not in any other situation.

To facilitate the second and third point, using a MPSC channel can be considered. Using MPSC would involve a smaller overhead than continuously using `SimulationElement.write`, as it would not require visiting each `Failure` every frame to ask whether it is active.

Some users might turn off all failures. Thus, the `Failure` type shall somehow receive this piece of configuration information and ignore any failures triggered by the element. As cross-environment configuration hasn't received much attention yet, for now we'll just assume failures are always allowed to occur.
