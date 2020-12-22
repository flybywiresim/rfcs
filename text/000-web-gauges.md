- Feature Name: `web-gauges`
- Start Date: 2020-12-22
- RFC PR: [flybywiresim/rfcs#0000](https://github.com/flybywiresim/rfcs/pull/7)
- ~~A32NX Issue: N/A~~

# Summary
[summary]: #summary

The RFC concerns a new framework by which the presentational portion of gauges, and possibly other components, can be run outside of the simulator. The proposition includes a websocket communication interface between the running aircraft logic inside the simulator and gauges outside of the simulator, over a local network owned by the user. The gauges can then be used on anything that has access to that network and has a web browser.

# Motivation
[motivation]: #motivation

The motiviation for this feature originate from two desires:

1. Developers would like a better way to develop and test gauges. **Currently, any change made to gauges (and any of the project code for that matter) requires the developer to restart the simulator to effect the changes.** This is a cumbersome process, and a faster way to develop gauges, especially their presentational code, will save much time and provide for a far greater developer experience. Running the gauges, which are effectively web applications, outside of the simulator means the developer can leverage common web development workflows which are fast and have wide debugging capability.
2. Whilst the simuator is able to achieve incredible levels of realism, **interacting with smaller and more complex controls around the aircraft can be unrealistic and challenging using pointer devices.** Allowing for gauges to be run and used outside of the simulator, controls and gauges can then be used on touch devices such as tablets and mobile phones, which increases the realism and ease of use. Realism in the simulator is also reduced by the limited viewport offered by a single monitor. Adding the ability for gauges to be used on more than one device allows users to increase their screen real estate used for the simulator, improving realism and immersion.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

**Web Gauges** can be run inside and outside of the simulator. When run outside of the simulator, they communicate over the network by means of a websocket connection. Each web gauge has a websocket adaptor which handles the connection with the websocket server. When run inside of the simulator, they communicate directly with the surrounding logic. The server is fed with data it needs to send to web gauges, and receives events sent by the web gauges. As such, each gauge type is required to be defined and understood by the simulator-side logic (i.e. you cannot run your own custom gauge type outside of the simulator that has not been defined in the websocket server and surrounding logic).

## Example: MCDU

A great example for this architecture is the MCDU. The MCDU comprises of a screen which renders lines of text. It also has buttons which are pressed, sending press events to the controlling logic. The portion of the MCDU which would be placed into a web gauge component would be simply the presentational code that renders the text on the screen. Any other logic should be kept in the simulator unless it is strictly limited to presentational output.

In the simulator, the MCDU also includes a part of the aircraft model (buttons and surrounding frame). The mandatory scope of this is situated outside of this new web gauge framework. However, in the MCDU case, the external version of the web gauge might render a 2D version of the entire MCDU hardware, with buttons and status lights. For these cases, the internal model and external web gauge project can make extended use of the websocket interface to include these parts of the gauge so that events can be fired from button presses outside of the simulator and the simulator-side code handling the button presses, and status lights receiving data from the simulator.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

![webgauge framework](https://user-images.githubusercontent.com/8268040/102900973-f2736800-4464-11eb-81c7-7a66e24f0819.png)

The framework is designed to attempt to achieve the following core characteristics:

* Clear delineation between controlling logic and presentational code.
* The ability to share the presentational code at its point of implementation inside the simulator, as well as standalone as the web guage externally (by share of code, I mean using the code exported as a module consumed both places)

The web gauge framework consists of the following:

* **Gauge:** The template and template-related code to render the gauge. It should take in values provided by the controller and render them appropriately. The actual data schema is completely up to the gauge developer, and can be as low-level or as abstracted as they wish. For example, this would be the MCDU screen.
* **Webgauge Controller:** The logic that either is or sits in front of operational logic running the gauge. This is the code that interacts with the simulated aircraft and simulator framework. It sends data to the gauge to effect visual updates, as well as receives events from the gauge, handling them appropriately.
* **Websocket Server:** This is the server which will allow incoming socket connection requests from outside of the simulator. The connection specifics can be configured in an options page on the MCDU, much the same way Simbrief is configured.
  * Connections will be namespaced by gauge type. For example, if the simulator aircraft offers an `mcdu` gauge type, an incoming connection can ask to subscribe to the `mcdu` socket namespace, which means it will receive values from the `mcdu` webgauge controller. It will also be able to send `mcdu` events, which will eventually be handled by the `mcdu` controller. Namespacing in websocket communication is achieved in many different ways depending on the abstraction used. For example, `socket.io` implements namespacing [as described](https://socket.io/docs/v3/namespaces/).
* **Webguage Socket Adapter:** Inside the simulator, the models and gauges will communicate "directly" with the webguage controller. Outside of the simulator, they will need to do this via the websocket connection. The webguage socket adapter will take websocket messages and map them into an interface which matches the webgauge controller. This means that we can keep the websocket code standard, as well as make the gauge implementations situationally agnostic.

I've intentionally kept the socket architecture quite flexible. This leaves room for many extensions well outside the scope of this RFC motivation which I cover in the [Future Possibilities](#future-possibilities).

# Drawbacks
[drawbacks]: #drawbacks

- Increased system breadth. This is not the same as complexity since the overriding characteristic of the above architecture introduces organisation and separation of concerns, which has spinoff maintainability improvements in the future. System breadth can have the reverse effect depending on how well the architecture is documented.
- Inability to locally debug operational logic. If it is decided that in fact the web gauge framework is to be scoped only to presenational code, then the existing issues around the debugging workflow remain for operational logic in that it will still only run in the simulator.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

> Why is this design the best in the space of possible designs?

I'm willing to leave this open for those who might know the current architecture better, as well as for those who may actually want to leverage it differently. To me, the positioning of the scope boundary makes the most sense, as well as covers most of the problem domain.

> What other designs have been considered and what is the rationale for not choosing them?

No other communication protocols have been considered. Websockets is the most obvious option for full-duplex structured data communication as well as ease of implementation.

> What is the impact of not doing this?

None. A32NX still works incredibly well; this is purely an enhancement to make developers happy and mouse-users even happier.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- How much of the above archtectural separation of concerns (between operational and presenational code) already exists? If not, how deeply mixed are the two, and how much effort would it require to extricate the two?

# Future possibilities
[future-possibilities]: #future-possibilities

- A network interface for more than just webgauges. This could be a starting point for communication of real hardware with the A32NX mod, or control-only web interfaces.
- Distribution of operational logic. It's possible that the boundary for internal/external scope can be shifted so as to possibly host much of the operational logic outside of the simulator for debugging or even performance. The interface for these would need to be carefully considered because as soon as you distribute full-duplex logic, you need to start considering parallel characteristics and practices.
