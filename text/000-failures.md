# Summary

Whether it's the thrill of flying without knowing when failures will hit you, or the ability to test your skills in a failure scenario you set up; equipment failures add an exciting level of simulation on top of what we've built thus far.

This document discusses some of the functionality we could provide and provides a software design for its foundation.

# Requirements

*Equipment* is some piece of the aircraft, whether hard- or software, which is necessary for its operation.

A *Failure* is the abnormal functioning of a piece of Equipment.

A *Trigger* is a cause or means through which a failure comes into existence.

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

>  I've been thinking about a failure setup that is a bit more gamified and orchestrated by a director - similar to games like Left 4 Dead and Rimworld, where a director is in charge orchestrating the failures without the user having to select specific systems. For example:
> - Failing a random number systems when starting the aircraft but making sure to stay compliant to the MEL ("Let's see how challenging our flight will be today", without causing an emergency or the flight to end immediately alone).
> - Failing systems unpredictably one by one to eventually reach a pre-coded scenario with a very specific failure configuration (things that would be very, very rare in reality because of redundancy, but are interesting - e.g. multi ADR failure).
> - Having the omniscient director take into account factors like distance to nearest airport, equi-time points or airport equipment to selectively fail important systems for that flight phase to make a challenging, but salvageable situation (maybe the user can select the difficulty).
>
> The idea here is that it's more about indicating to the aircraft "I'd like an unexpected/tough/basically-impossible situation" without having to code things one by one. I'm thinking this system would be an overlay on top of the failure system itself.

Thus, a *Director* would essentially provide a predefined or generated script (also called a *Scenario*), which triggers failures. We'll not get ahead of ourselves, but this is interesting stuff to add indeed!

The failures, triggers, director and scenarios are configured through the EFB. Note that this means that making the EFB a piece of failable equipment therefore might not be the best idea. 😉

If at any point in the future we store data on the aircraft which we can reload on the next flight, then failures should also be stored. Similarly, any stress-based information should also be stored.

# Design

The configuration of failures happens through the EFB. This doesn't necessarily mean the responsibility of triggering failures (lets call it orchestrator for now) should also lie there, although it is the most likely candidate.

One of the main problems we face is having our equipment declared in multiple languages and environments. Systems are in a Rust built WASM, other systems and computers are in JS running within various displays, autopilot and FADEC are C++ built WASM. One would preferably not know from the perspective of the orchestrator which piece of software executes the failure.

The communication protocol is also highly limited, with some parts only being able to communicate floating point numbers.

## Communication

Consider the orchestrator to be the publisher, and the various environments (Rust WASM, C++ WASM, JS per screen) consumers. 

### Most triggers

Each failure has its own numeric identifier (*F<sub>i</sub>*), which is known both by the publisher and exactly one of the consumers. The publisher has a queue of failures. For each failure in the queue the publisher writes *F<sub>i</sub>* to a SimVar *SV<sub>fail</sub>*. Consumers read *SV<sub>fail</sub>* and determine if they are responsible for handling *F<sub>i</sub>*. If so, they execute the failure within their environment and overwrite *SV<sub>fail</sub>* with *0*. If not, they leave *SV<sub>fail</sub>* as is. When the queue contains another identifier, the publisher keeps observing *SV<sub>fail</sub>*, waiting for it to become *0* and then writes the next identifier. It is important to note that SimVar writes are asynchronous, and thus a SimVar's written value might not be visible on the next frame. Therefore, the publisher should take care not to write values too quickly.

Failure repair involves another SimVar *SV<sub>repair</sub>* through which *F<sub>i</sub>* may be communicated to repair the failure. Here I specifically use *F<sub>i</sub>* instead of *0* (fail) and *1* (repair), due to the asynchronous nature mentioned above. If instead of having one SimVar for failures and one SimVar for repairs we had one SimVar for *F<sub>i</sub>* and another for a boolean indicating failure or repair, then there would be no means through which we would with certainty determine that the pair *(F<sub>i</sub>, bool)* reflects the desired command.

This communication design is simple and should work well enough for most cases. The real-world failure rate and stress triggers are exceptions to this. 

### Real-world failure rate trigger

For real-world failure rate we probably ought to accept that the failure rates are defined within the orchestrator, and thus will be re-defined per aircraft, even though they might contain the exact same equipment.

### Stress/misuse trigger

Stress/misuse triggers are special as only the equipment itself can detect the abuse. Thus, we cannot give this responsibility to the orchestrator. However, the orchestrator does need to be aware of the failure as otherwise there is no way for it to indicate the failed equipment to the user and thus the user wouldn't be able to repair the failure.

Due to the asynchronous problem described above and the fact that this case has multiple publishers and a single consumer, simply writing another SimVar with *F<sub>i</sub>* leads to problems. Thus we'll probably need to introduce two additional SimVars: *SV<sub>lock</sub>* to which an environment writes its own unique identifier, and *SV<sub>misuse</sub>* to which the environment writes *F<sub>i</sub>* after the lock has been acquired, which can be verified by checking if *SV<sub>lock</sub>* contains the unique identifier of the environment.

The consumer will read *SV<sub>misuse</sub>* and overwrite it with *0* after reading. The publisher itself will then either remove the lock written to *SV<sub>lock</sub>* or write another *F<sub>i</sub>* into *SV<sub>misuse</sub>*. The responsibility of removing the lock remains with the publisher, as giving this responsibility to the consumer could lead to a situation where the publisher reads *SV<sub>lock</sub>* and *SV<sub>misuse</sub>*, with the following result: *(ID, 0)*, and then incorrectly assumes it can write the next *F<sub>i</sub>*, while in fact *SV<sub>lock</sub>* has become *0* right after the read.
