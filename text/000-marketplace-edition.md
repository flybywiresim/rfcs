- Feature Name: a32nx_marketplace_edition
- Start Date: 2021-06-01
- RFC PR: [flybywiresim/rfcs#12](https://github.com/flybywiresim/rfcs/pull/12)

# Summary
[summary]: #summary

This establishes the nature, purpose, goal and maintainenance requirements of the A32NX Marketplace Edition repository, referred to here as A32NX:ME. 

# Motivation
[motivation]: #motivation

There have been many concerns raised with the inclusion of the A32NX into the in-sim marketplace. To name a few:

1. Compatibility of online features (currently: TELEX, WX Request, SimBrief integration, live map) on XBOX
2. Licensing concerns - MS does not want packages using a copyleft license on the Marketplace
3. Bureaucratic concerns - MS plays safe with model changes, requiring Airbus to appreove every model edition or addition, since the in-sim markeptlace becomes an officially endorsed item

It has been decided with a large majority (11 strongly in favour, 6 in favour, 4 without an opinion, 2 against), measured using a team-wide survey, to separate the A32NX into mainline and marketplace editions.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

In order to directly address the concerns listed above, the following steps will be taken:

- Online feautres (listed in point 1) will **not** make their way to **A32NX:ME**; (point 1); 
- The **mainline** edition will revert its license back to GPLv3 (point 2);
- **A32NX:ME** will constitute a separate repository licensed under the MIT license (point 2);
- Major additions, like model variants (A321, A319) will **not** make their way to **A32NX:ME** (point 3);

To ensure the robustness of the protection offered by the GPLv3 license chosen for the **mainline** edition AND to limit maintenance work, a limited set of future improvements will be ported
to **A32NX:ME**, detailed [below](#chosen-transfered-features).

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Chosen transfered features
[chosen-transfered-features]: #chosen-transfered-features

A survey has been conducted in the FBW development team to find out which future improvements and additions are considered **essential** for transfer in **A32NX:ME**.

Here are the results:

```
My preference would be to have everything in there; i.e., no separate editions. That not being possible, then my preference would be for it to have everything that each developer of a feature does not consider it needing to be under the MIT license. Since I don't code, I don't really have a dog in this fight, so officially my response is "I don't care."
```

```
Abandon Marketplace
```

```
Basic LNAV/VNAV/FWC implementation, custom AP+FBW, flyPad OS v2
```

```
LNAV (although accept this is not “basic”)
```

```
Absolutely none!
```

```
Whatever is in 0.7 but in very very basic state.
```

```
Small LNAV and VNAV implementation (not complete, just basic)
```

```
LNAV flying STARS properly should be enough. If we can fix the default FPM by bringing in Asobo’s newer code with minimal effort that would be ideal. 
```

```
Simple (linear) engine start-up and shutdown
```

```
A page on the cockpit door cam which switches to Icemans face at the push of a button. In all seriousness though I think the next marketplace update should be when LNAV and the FPM are implemented whenever that may be 
```

```
LNAV, VNAV, hydraulics, pneumatic/bleed system, pax loading on EFB, EFB flyPadOS v2, FADEC startup/shutdown, QNH STD blinking on PFD, AP OFF and ATHR off warnings on upper ECAM, ILS tuning issues, performance improvements
```

```
Nothing!
```

## Points of action

In order.

### License switch of mainline edition

The following steps must be taken in a reasonable amount of time in order to accomplish point 2:

- PRs which match the list outline in TODO must be marked with the "Needed for ME Fork" label and later merged;
- Consent must be obtained from contributors who have added MIT-licensed code to the `flybywiresim/a32nx` repository to relicense their code as GPLv3;
- When that is accomplished, the license switch PR must be created and merged, and a notice signaling the different location of the **A32NX:ME** sources must be added to the README.

### Creation of the `flybywiresim/a32nx-marketplace-edition` repository

- The above repository must be created, based on the tip of the `v0.6` branch of `flybywiresim/a32nx` (done);
- CI actions must be setup to generate marketplace builds for both publishing and QA.

### Removal of online features from A32NX:ME

The following features will have to be removed from **A32NX:ME**:

* TELEX (free text);
* WX Request (METAR/TAF, ATIS);
* SimBrief integration;
* Live map.

*Note: if any online features are later added to the **mainline** edition, they **must not** make their way to **A32NX:ME***

## Long-term responsibilities

### Transfer of features

*Note: chose transfered features are listed [above](#chosen-transfered-features)*

### Maintenance of **A32NX:ME** repository

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
