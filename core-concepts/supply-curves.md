---
description: Learn more about Doppler's custom supply curves
icon: chart-waterfall
---

# Supply curves

## Supply Curves

In Doppler, **supply curves** define how token price changes relative to the amount sold during a price discovery auction. Instead of a single fixed price, a supply curve maps cumulative tokens sold to price, allowing controlled, potentially predictable price progression as demand unfolds.

***

### What a Supply Curve Does

* Determines **how price increases** as tokens are purchased
* Shapes **auction dynamics** (early cheap access vs. steep climbs)
* Signals **scarcity and demand** to participants
* Influences **liquidity migration outcomes**

A well-designed curve aligns economic incentives and supports efficient market formation.

***

### Curve Types Supported

Doppler supports multiple supply curve models to fit different launch goals:

#### Static Supply Curves

Flat or linear progression with a single supply curve.\
Price increments are uniform as tokens sell.

> Useful for simple, predictable price growth without complex structure.

***

#### Multiple Supply Curves

Multiple segments of the same market with distinct slopes.\
Each segment can have a different price rate.

> Lets projects encode phases (e.g., early discount → steeper later pricing).

***

#### Dynamic / Time-Based Supply Curves

Curve parameters evolve over time or based on demand signals.\
Price can accelerate or decelerate dynamically.

> Ideal for launches that need tempo-aware pricing behavior or maximum efficiency.

***

### Designing Good Curves

* Gentle slope → smoother, more gradual price movement
* Steep early slope → faster price growth, tighter early access

The choice of curve impacts:

* Token distribution speed
* Participant experience
* Initial liquidity depth

Supply curves are a core economic primitive in Doppler - they convert demand into predictable pricing, designed to be easily updatable and enabling iterations towards an ideal market structure.

***

### When to Choose Which Curve

* **Static** → Simple, predictable auction
* **Multicurve** → Phased launches with differentiated pricing. Well suited for low value assets.
  * Multicurve is considered a _strict_ improvement over static due its designs ability to implement a superset of static's functionality and should be used on supported networks.
* **Dynamic** → Demand/time-sensitive markets. Well suited for high value assets.

Custom supply curves offer flexibility for projects to design markets tailored to their use cases, without custom contracts - all defined and adjustable in simple configurations at launch time.
