# Design Decisions

## 1. Purpose

This document explains several important design decisions behind **Elden Thing**, a Java object-oriented game built on top of the FIT2099 game engine provided by Monash University.

The goal is not to reproduce the original assignment rationale or assessment solution.

Instead, this document provides a retrospective engineering explanation of:

* what problem each design addressed;
* why a particular abstraction was useful;
* what alternative approaches were possible;
* what trade-offs were introduced;
* what I learned from the decision.

---

# 2. Extend the Existing Engine Instead of Rebuilding It

## Problem

The project already had a game engine providing abstractions such as:

```text
Actor
Action
Item
Ground
Location
GameMap
World
```

A simple approach would have been to ignore some of these abstractions and build separate systems for new requirements.

That would create two competing architectures:

```text
FIT2099 Engine

+

Custom Parallel Game Framework
```

which would make integration increasingly difficult.

## Decision

Game-specific functionality was designed around the extension points already provided by the engine.

Conceptually:

```text
FIT2099 Engine
      ↓
Existing Extension Points
      ↓
Game-Specific Classes
      ↓
New Gameplay Features
```

The project-specific implementation remained primarily inside the `game` layer.

## Why

This approach reduced unnecessary duplication and allowed new systems to participate naturally in the engine's existing game loop.

For example:

* new characters extend actor concepts;
* new interactions become actions;
* new terrain extends ground concepts;
* world navigation reuses `GameMap` and movement abstractions.

## Alternative

Another approach would have been to create custom controllers for every new subsystem.

For example:

```text
CombatController
FarmingController
TeleportController
NPCController
```

This may initially appear simpler, but it would bypass many of the engine's existing extension mechanisms.

## Trade-off

Working with the engine required more time initially because its architecture had to be understood before implementing features.

However, that upfront cost made later integration more consistent.

## Lesson

One of the most important lessons from this project was:

> Before adding a new abstraction, first check whether the existing framework already provides an appropriate extension point.

---

# 3. Represent Interactions as Actions

## Problem

As the number of game interactions increased, placing every operation directly inside `Player` or NPC classes would create very large domain objects.

A player might otherwise need methods such as:

```text
attack()
buy()
sell()
plant()
eat()
cure()
refill()
prophecise()
answerRiddle()
...
```

This would continuously increase the responsibilities of the Actor classes.

## Decision

Interactions were represented through specialised Action objects.

Examples from the final implementation include concepts such as:

```text
AttackAction
BuyAction
SellAction
PlantAction
GrowAction
EatAction
CureAction
ReproduceAction

PropheciseAction
RiddleAnswerAction

RefillWateringCanAction
RefillSprinklerAction
```

## Why

An Action represents a single executable interaction.

This creates the separation:

```text
Actor
  ↓
available actions
  ↓
selected Action
  ↓
operation executed
```

An actor therefore does not need to directly implement every interaction available in the world.

## Alternative

A straightforward alternative would be a large conditional structure:

```java
if (choice == ATTACK) {
    ...
} else if (choice == BUY) {
    ...
} else if (choice == PLANT) {
    ...
}
```

As requirements grow, this becomes harder to maintain.

## Trade-off

Action-based design creates more classes.

A small game might require fewer files if all behaviour were placed directly inside actors.

However, once the number of interactions grows, the extra classes provide clearer separation of responsibilities.

## Lesson

More classes are not automatically overengineering.

When each class represents a meaningful changing behaviour, separating actions can reduce coupling and make later requirements easier to introduce.

---

# 4. Separate Behaviour from Action

## Problem

NPCs need to make decisions automatically.

There are actually two different problems:

```text
What should the NPC do?

vs.

How is that operation executed?
```

Combining both inside a single NPC class would couple AI decision logic to game mechanics.

## Decision

The project separates:

```text
Behaviour
    ↓
decision / intention

Action
    ↓
execution
```

Examples of behaviours include:

```text
AttackBehaviour
FollowBehaviour
WanderBehaviour
ReproduceBehaviour
GrowBehaviour
DeathBehaviour
MonologueBehaviour
```

A behaviour can determine an appropriate action without implementing the entire action itself.

## Why

This allows different NPCs to reuse the same underlying game operations.

For example:

```text
AttackBehaviour
      ↓
AttackAction
```

The behaviour decides that attacking is appropriate.

The action performs the attack.

## Alternative

Each NPC could contain its own logic:

```text
OmenSheep.turn()
SpiritGoat.turn()
GoldenBeetle.turn()
Guts.turn()
```

with every class independently implementing decision logic.

That would likely duplicate logic and make changes to common behaviours harder.

## Trade-off

Separating behaviour and action introduces another abstraction layer.

Understanding the execution flow therefore requires following more than one object.

However, the separation becomes valuable when the same actions and behaviours are reused across several actor types.

## Lesson

This decision helped clarify an important architectural idea:

> Decision-making and execution are separate responsibilities.

---

# 5. Make NPC Decision Algorithms Interchangeable

## Problem

Even after behaviours were separated from NPC classes, another question remained:

> If several behaviours are available, how should the NPC choose one?

Hard-coding one selection algorithm into every NPC would make experimenting with alternative behaviour models difficult.

## Decision

Behaviour selection was abstracted behind:

```text
BehaviourSelectionStrategy
```

with implementations including:

```text
PriorityBasedStrategy
RandomSelectionStrategy
```

Conceptually:

```text
NPC
 ↓
Available Behaviours
 ↓
BehaviourSelectionStrategy
 ├── Priority Based
 └── Random
 ↓
Selected Behaviour
```

## Why

The same creature can use different decision algorithms without requiring a different creature implementation.

The application setup even creates examples of the same creature type using different strategies.

## Alternative

The selection logic could have been embedded directly inside each NPC:

```java
if (canAttack) ...
else if (canFollow) ...
else wander ...
```

This would work but would tightly couple each creature to one decision algorithm.

## Trade-off

The Strategy Pattern introduces interfaces and additional implementations.

If only one selection algorithm ever existed, that abstraction would provide limited value.

In this project, multiple strategies actually exist, so the variation is real rather than hypothetical.

## Lesson

A useful Strategy abstraction should represent a behaviour that genuinely varies.

Using a design pattern only because it exists would add complexity without solving a real problem.

---

# 6. Represent Capabilities with Interfaces

## Problem

Some game entities share individual capabilities without logically belonging to the same concrete inheritance hierarchy.

For example, an object might be:

* reproducible;
* hostile;
* harvestable.

Those characteristics do not necessarily describe what the object *is*.

They describe what the object *can do*.

## Decision

Capabilities were represented through abstractions such as:

```text
Reproducible
HostileAttacker
Harvestable
```

## Why

Game logic can depend on a capability instead of a concrete implementation.

Conceptually:

```text
Can this object reproduce?

instead of

Is this object an OmenSheep?
```

This reduces unnecessary knowledge of concrete types.

## Alternative

The code could repeatedly check concrete classes:

```java
if (actor instanceof OmenSheep ||
    actor instanceof SpiritGoat) {
    ...
}
```

This would require the existing logic to change every time another reproducible creature was introduced.

## Trade-off

Capability interfaces increase the number of abstractions.

They are most valuable when multiple unrelated classes need the same behaviour.

## Lesson

Inheritance answers:

> What kind of object is this?

Interfaces can instead answer:

> What can this object do?

That distinction became increasingly useful as the game domain grew.

---

# 7. Model Plants Through a Shared Lifecycle Abstraction

## Problem

The farming system includes different plant types with both shared and plant-specific behaviour.

The final implementation includes concepts such as:

```text
Plant
├── InheritreePlant
└── BloodrosePlant
```

Both plants need concepts such as:

* growth;
* hydration;
* lifecycle;
* dehydration.

But individual plant types may have different timing and outcomes.

## Decision

Shared plant functionality was placed in a common Plant abstraction, with specialised subclasses defining plant-specific behaviour.

## Why

Without a common abstraction, each plant implementation could duplicate lifecycle logic.

Conceptually:

```text
Plant
 ├── common lifecycle state
 ├── common hydration concepts
 └── shared behaviour

        ↓

Specific Plant
 └── plant-specific rules
```

## Alternative

Each plant could be implemented independently:

```text
InheritreePlant
BloodrosePlant
FuturePlantA
FuturePlantB
```

with no common parent.

This may initially be simpler but would likely repeat state-management logic.

## Trade-off

A base class creates inheritance coupling.

If plant types eventually became radically different, excessive functionality in the parent class could become difficult to maintain.

Therefore only genuinely shared plant responsibilities should remain in the common abstraction.

## Lesson

Inheritance is most useful when there is a real shared lifecycle or invariant, not merely because two classes are conceptually related.

---

# 8. Separate Watering Devices from Plants

## Problem

Plants need water, but there are multiple ways to supply it.

The project contains both manual and automatic watering mechanisms.

A naive approach would let each Plant know about:

```text
WateringCan
Sprinkler
FutureWateringTool
```

That would couple the plant lifecycle to specific equipment implementations.

## Decision

Watering was represented through a separate hierarchy:

```text
WateringDevice
      │
      ├── ManualWateringDevice
      │       └── WateringCan
      │
      └── AutomaticWateringDevice
              └── Sprinkler
```

## Why

This separates two questions:

```text
Plant
"What state is the plant in?"

WateringDevice
"How is water delivered?"
```

A plant therefore does not need to understand every tool capable of watering it.

## Alternative

Watering logic could have been embedded directly into individual plant classes.

For example:

```java
plant.waterWithCan(...)
plant.waterWithSprinkler(...)
```

Every additional device could then require changes to Plant.

## Trade-off

The device hierarchy creates additional abstractions.

But it gives a clearer boundary between plant lifecycle and equipment behaviour.

## Lesson

A domain entity should not know about every external mechanism that can modify it.

Separating those mechanisms can make both sides easier to extend.

---

# 9. Separate Oracle Dialogue Generation from the Oracle

## Problem

The Oracle supports several types of AI-generated interaction:

```text
Prophecy
Riddle
Compliment
```

Putting all prompt construction, response handling and API behaviour directly inside `Oracle` would give the NPC too many responsibilities.

## Decision

Dialogue generation was placed behind:

```text
DialogueGenerator
```

with implementations:

```text
ProphecyGenerator
RiddleGenerator
ComplimentGenerator
```

The final flow is conceptually:

```text
Oracle
  ↓
PropheciseAction
  ↓
DialogueGenerator
  ↓
ApiHandler
  ↓
External API
```

## Why

The Oracle should manage NPC interaction and Oracle-specific state.

It should not need to understand how every possible AI prompt is constructed.

Adding another dialogue mode can therefore be approached by introducing another generator implementation rather than heavily modifying the Oracle.

## Alternative

A single Oracle method could use a large switch:

```java
switch (dialogueType) {
    case PROPHECY:
        ...
    case RIDDLE:
        ...
    case COMPLIMENT:
        ...
}
```

Some selection logic still exists, but the detailed generation behaviour is delegated to separate classes.

## Trade-off

For only three dialogue modes, separate classes may appear more verbose.

However, AI integrations are likely to change independently, making the abstraction useful for isolating that variation.

## Lesson

The most useful abstraction point is often the part of the system that changes independently.

---

# 10. Keep External API Communication Behind a Boundary

## Problem

External APIs introduce infrastructure concerns that are different from game-domain concerns.

These include:

* HTTP requests;
* authentication;
* JSON formatting;
* streaming responses;
* error handling;
* response parsing.

Embedding this logic inside actors would tightly couple game entities to networking.

## Decision

External communication was isolated in an API utility layer.

Conceptually:

```text
Game Domain
    ↓
Dialogue Abstraction
    ↓
ApiHandler
    ↓
OpenAI API
```

## Why

The game layer can focus on:

```text
"I need a dialogue response."
```

while the API layer handles:

```text
"How do I communicate with the external service?"
```

## Alternative

`Oracle` itself could open an HTTP connection and process streaming responses.

That would mix:

* NPC state;
* dialogue selection;
* networking;
* JSON parsing;
* terminal streaming.

## Trade-off

The current API integration is still relatively lightweight and does not use a dedicated HTTP or JSON library.

A larger production system would probably benefit from a stronger API client abstraction and structured JSON parsing.

## Lesson

Even a small external integration benefits from a clear boundary between infrastructure and domain logic.

---

# 11. Turn AI Output into Application State

## Problem

Calling an LLM only to print generated dialogue would make the integration mostly cosmetic.

The Riddle feature required the AI output to affect future player interaction.

## Decision

Generated riddle information is stored in Oracle state.

The flow becomes:

```text
Generate Riddle
      ↓
Receive Response
      ↓
Parse Riddle Data
      ↓
Store Active Riddle
      ↓
Create Answer Actions
      ↓
Player Selects Answer
      ↓
Validate Result
```

## Why

This turns the external response into part of the game domain.

The generated content therefore survives beyond the original API call.

## Alternative

The game could simply display:

```text
"The Oracle asks..."
```

and then discard the response.

That would be easier to implement but would not create meaningful gameplay interaction.

## Trade-off

Stateful AI output creates additional concerns:

* response structure must be predictable;
* parsing can fail;
* state must remain consistent;
* external model behaviour is less deterministic than normal game logic.

## Lesson

Using AI effectively inside an application often means integrating the generated result into **normal application state**, rather than treating the model as an isolated chatbot.

---

# 12. Reuse the Existing Movement System for Multiple Maps

## Problem

Adding a second map requires some way of transporting actors between locations.

A completely separate teleportation framework could have been created.

## Decision

Map transitions reuse the existing game engine's movement abstractions.

The world includes areas such as:

```text
Valley of the Inheritree
        ↓
Teleportation
        ↓
Limveld
```

## Why

Movement already existed as an engine responsibility.

Teleportation therefore represents a different destination rather than an entirely different movement architecture.

## Alternative

A custom map manager could directly modify world and actor state.

That would duplicate functionality already available through the engine.

## Trade-off

Reusing engine actions means teleportation must fit the assumptions of the existing movement model.

However, the benefit is architectural consistency.

## Lesson

When a new feature is conceptually a specialised form of an existing operation, extending the existing abstraction is often better than building a parallel subsystem.

---

# 13. Use Composition for the Bed of Chaos

## Problem

The Bed of Chaos contains behaviour that can be represented through multiple interacting parts.

Putting all of its logic inside one Boss class would make that class increasingly complex.

## Decision

The boss was decomposed into concepts such as:

```text
BedOfChaos
    │
    ├── BedOfChaosPart
    ├── Branch
    ├── Leaf
    └── GrowablePart
```

## Why

This models the boss as a structure made from collaborating objects.

Different components can own their own behaviour rather than requiring the boss to manage every detail.

## Alternative

A single class could contain fields such as:

```text
branchCount
leafCount
growthState
attackState
...
```

along with all related behaviour.

This would reduce the number of classes but increase the complexity of the central boss object.

## Trade-off

Composition requires coordinating multiple objects.

It can therefore introduce more relationships to understand.

However, it avoids creating one extremely large entity with unrelated responsibilities.

## Lesson

Composition is particularly useful when an entity is naturally made up of independent parts with their own state or behaviour.

---

# 14. Prefer Real Variation Before Introducing Patterns

One of the broader lessons from the project is that design patterns should respond to real variation.

For example:

```text
BehaviourSelectionStrategy
```

is justified because the project actually has:

```text
PriorityBasedStrategy
RandomSelectionStrategy
```

Similarly:

```text
DialogueGenerator
```

has:

```text
ProphecyGenerator
RiddleGenerator
ComplimentGenerator
```

The abstraction exists because behaviour genuinely changes.

This is different from creating an interface for every class merely to claim that a design pattern has been used.

---

# 15. What I Would Improve Today

Looking back at the project, several areas could be improved further.

## External API Configuration

API configuration should ideally come from environment variables or configuration rather than a source-code constant.

A stronger design could resemble:

```text
Environment
    ↓
ApiConfiguration
    ↓
ApiClient
```

---

## Structured JSON Parsing

The current API layer performs relatively lightweight response extraction.

A production implementation would benefit from a JSON library and explicit response models.

For example:

```text
API Response
    ↓
JSON Parser
    ↓
RiddleResponse DTO
    ↓
Domain Riddle
```

This would make malformed responses easier to validate.

---

## Dependency Injection

Some dependencies are constructed directly.

A larger application could inject dependencies such as:

```text
DialogueGenerator
ApiClient
BehaviourSelectionStrategy
```

making components easier to test independently.

---

## Automated Testing

Because many systems are based on rules and state transitions, areas such as:

* behaviour selection;
* farming lifecycle;
* watering capacity;
* riddle parsing;
* trading rules;

would benefit from focused automated unit tests.

---

## API Failure Handling

External API failures should ideally degrade gracefully.

For example:

```text
API unavailable
      ↓
fallback dialogue
      ↓
game continues
```

rather than allowing an external service to negatively affect the main game loop.

---

# 16. Overall Design Takeaway

The biggest lesson from the project was that object-oriented design is not primarily about creating inheritance hierarchies.

The more important questions are:

```text
What responsibility is changing?

Who should own that responsibility?

Does another object need to know about this concrete implementation?

Can the variation be represented behind an abstraction?

Can I add the next requirement without rewriting unrelated code?
```

Working with the FIT2099 engine made these questions particularly visible because every new feature had to coexist with an architecture that already existed.

The project therefore became an exercise in:

```text
Understanding Existing Architecture
            ↓
Identifying Extension Points
            ↓
Separating Responsibilities
            ↓
Managing Variation
            ↓
Integrating New Requirements
```

rather than simply implementing individual game features.

---

# Source Code Notice

The complete source code is intentionally excluded from the public portfolio repository.

This document is a high-level retrospective of the engineering decisions demonstrated by the original implementation and does not reproduce the complete Monash University assessment solution.

---

## Related Documents

* [Project Overview](../README.md)
* [Architecture Overview](architecture.md)
* `reflection.md` — technical reflection and personal learning
