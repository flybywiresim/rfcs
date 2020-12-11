<!-- - Feature Name: (fill me in with a unique ident, `my_new_feature`)
- Start Date: (fill me in with today's date, 2020-12-11)
- RFC PR: [flybywiresim/rfcs#0000](https://github.com/flybywiresim/rfcs/pull/0000)
- A32NX Issue: [flybywiresim/a32nx#0000](https://github.com/flybywiresim/a32nx/issues/0000) -->

# Summary
[summary]: #summary

Creation of the systems stalled. The code itself contains useful algorithms, but the (reasoning behind the) design is unclear. This document intends to help to achieve a good systems software design in whatever language is used. The document itself uses Rust for code examples.

# Motivation
[motivation]: #motivation

Good software design makes implementing new features easier. While we cannot foresee every feature beforehand, going through the major features ensures a consistent design which is easy to work with and understand. In my opinion, good software design should primarily focus on defining the structural concepts that exist in the software. The amount of concepts should be  limited, as to not overburden those who develop within it with the continuous question of: "should I use concept x or y to do z?".

# Requirements

Before going through the requirements, some definitions:

* **System**: Does not refer to a software system but to a part of the aircraft, i.e. electrical system.
* **Model**: The part of the software that relates to actual parts of the aircraft, e.g. `ElectricalSystem`, `Engine`, `EngineGenerator`, etc.
* **Software**: The thing we're building 🤡.

## 1. Runs outside the simulator
The software should be testable outside of the simulator by running it in unit tests or a console application.

## 2. Simulator interactions outside the model
To aid in achieving requirement 1, interactions between the simulator and the model should be separated such that the model is unaware of the simulator's existence.

## 3. Guarantee consistent state
Each update of the model should leave the model in a consistent state. Requiring multiple "ticks" to reach the correct state is not acceptable. The programming model makes it easy to ensure this is guaranteed.

## 4. Observable state
A subset of systems requires observing the state of parts of the model, without the model itself having to be aware of such requirements. Two examples of such feature requirements are:

* The ability to play different sounds for the opening and closing of contactors in the electrical system.
* Showing faults and actions on the Lower ECAM.

## 5. Reuse in multiple Airbus aircraft types
Some parts of the model can be reused for multiple Airbus types. While the decision which contactors in the electrical system are opened and closed is a type specific decision, the fact that such an electrical system contains buses, contactors, generators, etc. is true for all aircraft.

## 6. Starting state for different phases of flight
The model should be easily initialisable to different stages of flight, including cold and dark, on the runway and in flight.

## 7. Tests are mandatory
Testing the software automatically is key to quick and relatively error-free development. Thus, unit and integration (not with MSFS but combining pieces of the model) testing must be fully supported and mandatory.

## 8. No confusion about units
The simulator exposes data in various forms, as pound, kilo, liter, gallon, volts, ampere, etc. This might lead to confusion during development and therefore should be mitigated.

# Design

## Runs outside the simulator
To handle this requirement, parts of the software that interact with the simulator should be located outside of the part of the software that models the systems. It is therefore an onion-like design.

To make this possible, I propose we use the visitor pattern:

```rust
fn main() {
    let mut airbus = A320::new();

    // Ignore boxing for the sake of example simplicity.
    let fromSimToModelVisitor = SimVarsFromSimulatorToModelVisitor {};
    airbus.accept(&fromSimToModelVisitor);

    // Performs the model calculations.
    airbus.update();

    let fromModelToSimVisitor = SimVarsFromModelToSimulatorVisitor {};
    airbus.accept(&fromSimToModelVisitor);
}

impl MutableVisitor for SimVarsFromSimulatorToModelVisitor {
    fn visit_engine(&self, engine: &mut Engine) {
        engine.n2 = simVars[format!("ENG N2 RPM:{}", engine.number)];
        // ...
    }

    // ...
}

// Note: we don't need a mutable visitor for reading from the model, but implementing
// both a mutable and immutable visitable for every type might be a bit of overkill.
// For a Rust implementation we might be able to macro it?
impl MutableVisitor for SimVarsFromModelToSimulatorVisitor {
    fn visit_engine_generator(&self, generator: &mut EngineGenerator) {
        simVars[format!("L:GEN{}_VOLTAGE", generator.number)] = generator.voltage;
    }

    // ...
}
```

## Simulator interactions outside the model
The above design also fulfills this requirement.

## Guarantee consistent state
Achieving a consistent state requires that any dependencies between calculations are clearly visible. For example, to determine how much power the engine generator produces we need to know the engine's N2. Thus, the state of the engine generator musn't be updated before the engine is updated.

The `update` function plays a key role in making these dependencies clear, as shown in the example below:
```rust
fn main() -> Result<(), Box<dyn Error>> {
    let mut airbus = A320::new();

    // We can add an UpdateContext type to provide data that is external to the system as
    // input for updating the systems.
    let update_context = UpdateContext {
        delta_time: Milliseconds(rng.gen_range(250, 1000)),
        ambient_temperature: Celsius(rng.gen_range(5.2, 10.8))
    };

    airbus.update(&update_context);
}

pub struct A320 {
    engine1: Engine,
    gen1: EngineGenerator,
    engine2: Engine,
    gen2: EngineGenerator,
    elec: A320ElectricalCircuit,
}

impl A320 {
    pub fn update(&mut self, context: &UpdateContext) {
        self.engine1.update(context);
        // As gen1 depends on data in engine1, engine1 must be updated before gen1.
        self.gen1.update(context, &self.engine1);
        self.engine2.update(context);
        self.gen2.update(context, &self.engine2);
        // As elec depends on data in gen1 and gen2, they must be updated before elec.
        self.elec.update(context, &self.gen1, &self.gen2);
    }
}
```

It is important to note that the `update` function is only allowed to modify the state of the type on which it is called, or any of the types owned by that type. Therefore, when passing `engine1` to `gen1` we provide an immutable borrow.

The number of `update` function parameters might increase quite a bit as the implementation grows. By setting the goal to keep this relatively low it will guide us in creating a good implementation structure. The `update` function above might trigger a question: should the `EngineGenerator`s be owned by the `A320ElectricalCircuit`?

On another note, the design above might make threading some of the calculations simpler. As long as we merely provide immutable borrows, we can use multiple threads to update parts of the system. One could for example run `engine1` + `gen1` updating in one thread, and `engine2` + `gen2` updating in another, then merge those threads and use the result in updating `elec`. Of course a downside of such a threading model is that the `A320` type becomes aware of threading, which somewhat mixes levels of abstraction.

## Observable state
The requirements mentioned two example features that we will implement in the future:

* The ability to play different sounds for the opening and closing of contactors in the electrical system.
* Showing faults and actions on the Lower ECAM.

My first thought on this was to introduce some form of publish/subscribe which would expose changes made within the model to the outside as events. However, this would introduce another concept and could also increases the number of responsibilities we give to the model itself. I mentioned in the [Motivation](#motivation) section that "the amount of concepts should be  limited". Therefore, I thus propose that we use the visitor pattern proposed in [Runs outside the simulator](#runs-outside-the-simulator) for this purpose as well.

Assuming the contactor sound is different when multiple contactors change state in one update, this might be implemented as:

```rust
fn main() {
    let mut airbus = A320::new();
    airbus.update(&update_context);

    // Ignore boxing for the sake of example simplicity.
    let electricalSoundVisitor = ElectricalSoundVisitor {};
    airbus.accept(&electricalSoundVisitor);

    // No idea how we'd run sounds in the sim...
    electricalSoundVisitor.queueSounds();
}

impl Visitor for ElectricalSoundVisitor {
    fn visit_contactor(&self, contactor: &Contactor) {
        if contactor.is_changed() {
            self.number_of_changed_contactors += 1;
        }
    }
}
```

The showing of faults and actions on the Lower ECAM could use a similar pattern, though of course it is larger and thus might be split into multiple visitors. By making the dependencies on the model read-only, we could also run these visitors on multiple threads, thus increasing performance if needed.

## Reuse in multiple Airbus aircraft types
By adhering to requirement 1 and 2, we can already try to implement parts of the A380 by composing types we created for the A320 in different ways. Certain minor differences, such as the cooling coefficient of an engine generator which might differ per type of engine can be implemented by providing them as input to the `new` (constructor) function.

## Starting state for different phases of flight
The visitor pattern can also be used to set the starting state for different phases of flight. Each phase of flight would be implemented as a separate `MutableVisitor` which would make calls on the model to start with the necessary system state.

```rust
fn main() {
    let mut airbus = A320::new();

    // Ignore boxing for the sake of example simplicity.
    let startingStateVisitor = match starting_state {
        StartingState::ColdAndDark => ColdAndDarkStartVisitor {},
        StartingState::InFlight => InFlightStartVisitor {},
        // etc.
    }

    airbus.accept(&startingStateVisitor);

    loop {
        // Update every frame or every x frames.
        airbus.update(&update_context);
    }
}
```

## Unit/integration tests
Unit tests by themselves need no additional thought as they can easily be implemented. Integration tests which combine parts of the model, or run on the whole model do require some more thought.

The integration tests will likely be about (1) having a system in a state, (2) applying some change to it (e.g. increasing the engine's N2) and then (3) observing the resulting model changes are as expected.

For (1), [Starting state for different phases of flight](#starting-state-for-different-phases-of-flight) provides a way to do so.

For (2), I see two options:

* Exposing properties of the system such that one can navigate through the system's model to change the value in the model. E.g. `a320.eng1.n2 = 123`.
* Creating visitors per (group of) tests which adapt a specific piece of the model.

I'm leaning towards (again) using visitors, as I'm quite sure the model's composition will change as we build more of the system. By picking the first option, those model changes would require changing all the tests that navigate through those piece of the model. By using a visitor we will not have that problem, as the visitor navigating through the model is the responsibility of the model.

However, if we follow the reasoning of the second option in (2), then (3) will also require visitors to execute assertions. ***If you have thoughts or ideas on this then please let me know.***

## No confusion about units
I suggest we use the [new type idiom](https://doc.rust-lang.org/rust-by-example/generics/new_types.html) to counter this, which would mean writing code like this:

```rust
struct Percentage(f64);

impl MutableVisitor for SimVarsFromSimulatorToModelVisitor {
    fn visit_engine(&self, engine: &mut Engine) {
        engine.n2 = Percentage(simVars[format!("ENG N2 RPM:{}", engine.number)]);
    }
}
```

<!-- # Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

A quick overview: explain the feature/change as if it were already implemented in the aircraft's code, and you were teaching it to another developer on the project. 

Introduce new named concepts, use examples when applicable, etc.

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

How this feature or change could be expanded on in future PR's and RFC's. -->
