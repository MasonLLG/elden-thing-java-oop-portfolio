# Technical Reflection

## 1. Introduction

Elden Thing was one of my first projects where the main challenge was not simply implementing a list of features.

The project was built on top of an existing Java game engine provided by Monash University, which meant that before adding new functionality, I first had to understand how an existing object-oriented system was structured.

This changed the way I thought about software development.

Earlier, I mainly viewed object-oriented programming as:

* creating classes;
* using inheritance;
* encapsulating fields;
* writing methods around objects.

During this project, I started to realise that the more important questions were:

> Who should own this responsibility?

> Which parts of the system are likely to change independently?

> How can a new requirement be added without rewriting unrelated classes?

> Should this relationship use inheritance, composition, or an interface?

> Am I extending the existing architecture or accidentally building a second architecture beside it?

These questions became increasingly important as the project grew.

---

# 2. Learning to Work with an Existing Codebase

One of the most valuable parts of the project was working with a framework that I did not design myself.

The FIT2099 engine already provided concepts such as:

```text
Actor
Action
Item
Ground
Location
GameMap
World
```

At the beginning, it was tempting to focus immediately on implementing new features.

However, I gradually learned that doing so without first understanding the framework could easily lead to unnecessary complexity.

The more effective process became:

```text
Read Existing Code
       ↓
Understand Responsibilities
       ↓
Identify Extension Points
       ↓
Design Feature
       ↓
Integrate with Framework
```

For example, when implementing new interactions, it was usually better to extend the existing `Action` model than to create an independent interaction system.

Similarly, moving between maps could reuse existing movement abstractions instead of introducing a separate world-navigation framework.

This was one of my first practical experiences with an important reality of software engineering:

> Most professional software development involves extending existing systems rather than building everything from scratch.

---

# 3. My Understanding of OOP Changed

Before this project, I associated object-oriented design heavily with inheritance.

A common way I thought about a system was:

```text
Base Class
    ↓
Subclass
    ↓
More Specialised Subclass
```

During the project, I started seeing the limitations of relying too heavily on inheritance.

Some characteristics are not really about what an object **is**.

They are about what an object **can do**.

For example:

```text
Reproducible
HostileAttacker
Harvestable
```

These capabilities can apply to different types of objects without requiring them to share one deep inheritance hierarchy.

That helped me understand why interfaces can sometimes represent the domain more naturally than inheritance.

My thinking gradually shifted from:

> "What class should this inherit from?"

to:

> "What responsibility or capability am I trying to represent?"

---

# 4. Separating Behaviour from Execution

The NPC system was particularly useful for understanding responsibility separation.

An NPC needs to decide what it wants to do.

But deciding to attack is not the same responsibility as performing an attack.

This led to a conceptual separation:

```text
Behaviour
    ↓
"What should I do?"

Action
    ↓
"Execute the operation"
```

For example:

```text
AttackBehaviour
      ↓
 AttackAction
```

The Behaviour decides whether attacking is appropriate.

The Action performs the actual interaction.

This helped me see that two pieces of code can be closely related while still representing different responsibilities.

If the two responsibilities change for different reasons, separating them can make the design easier to maintain.

---

# 5. Understanding the Strategy Pattern Through a Real Problem

Before using design patterns in larger projects, patterns sometimes felt like definitions to memorise.

For example:

> Strategy Pattern allows algorithms to be interchangeable.

That definition was easy to understand theoretically, but the value became much clearer when the project required different NPC decision-making approaches.

The project supported:

```text
BehaviourSelectionStrategy
├── PriorityBasedStrategy
└── RandomSelectionStrategy
```

The important part was not that the code contained a Strategy Pattern.

The important part was that there was a real point of variation:

```text
Same NPC
   +
Different Behaviour Selection Algorithm
```

The NPC itself did not need to change just because the algorithm used to select behaviours changed.

This made me understand a broader principle:

> A design pattern is useful when it isolates something that genuinely varies.

Using patterns only because they are known patterns can create unnecessary abstraction.

---

# 6. Composition Became More Meaningful

The Bed of Chaos implementation also helped me better understand composition.

A complex entity could have been represented as one large class containing all boss mechanics.

Conceptually:

```text
BedOfChaos
├── branches
├── leaves
├── growth
├── attacks
├── state
└── other behaviour
```

Instead, the boss can be represented using collaborating parts.

```text
BedOfChaos
    │
    ├── Branch
    ├── Leaf
    └── GrowablePart
```

This made the concept of:

> favour composition where appropriate

more concrete.

Composition is not automatically better than inheritance, but it can be useful when an entity is naturally made from smaller parts with their own responsibilities.

---

# 7. Modelling a Lifecycle Was Different from Implementing CRUD

The farming system was also an interesting design problem because plants change over time.

A plant is not simply created and stored.

It progresses through states.

Conceptually:

```text
Seed
 ↓
Plant
 ↓
Growth
 ↓
Watering
 ↓
Maturity
 ↓
Harvest
```

or, if hydration requirements are not satisfied:

```text
Growing Plant
     ↓
Dehydration
     ↓
Withered Soil
```

This helped me think more carefully about **stateful domain logic**.

The important question was not only:

> What data does a plant contain?

but also:

> What transitions are valid over the lifetime of the object?

This kind of thinking later became useful when working with other systems involving statuses, workflows and state transitions.

---

# 8. Separating Plants from Watering Devices

The watering system also changed the way I thought about dependencies.

A simple design could allow plants to directly understand:

```text
WateringCan
Sprinkler
```

But this would mean every time a new watering mechanism was added, plant classes might also need to change.

Instead, watering was treated as a separate concept.

```text
Plant
"What state am I in?"

WateringDevice
"How is water supplied?"
```

This helped me understand the value of separating:

* domain state;
* mechanisms that operate on that state.

The benefit becomes especially clear when multiple implementations exist.

---

# 9. External API Integration Was More Than an HTTP Request

The Oracle was my first experience integrating an LLM-style external API directly into game behaviour.

At first glance, external API integration appears to be mainly about:

```text
Send Request
     ↓
Receive Response
```

But the architectural problem was larger.

The system needed to separate:

```text
Oracle
    → NPC interaction

DialogueGenerator
    → what kind of dialogue is needed

ApiHandler
    → how external communication works
```

This made me think more carefully about infrastructure boundaries.

The Oracle should not need to understand HTTP request details.

Likewise, the networking code should not need to understand all game rules.

---

# 10. AI Output as Application State

The Riddle feature was especially useful because the generated response was not simply printed and discarded.

The system needed to take generated content and turn it into persistent interaction state.

Conceptually:

```text
LLM Response
     ↓
Parse Riddle
     ↓
Store in Oracle
     ↓
Generate Player Actions
     ↓
Player Answer
     ↓
Validate
```

This was an important distinction.

Instead of treating the LLM as:

```text
User → Chatbot → Text
```

the API became part of a normal application workflow.

That helped me understand a principle that I still find useful:

> AI-generated content becomes much more interesting when it participates in normal application state and business logic.

---

# 11. What Became Difficult as the Project Grew

As more requirements were added, I noticed that design decisions made earlier became increasingly important.

A tightly coupled design might work well for the first requirement.

The problem often appears later.

For example, if every NPC contained its own behaviour-selection logic, introducing additional NPC behaviour strategies would require changes across multiple classes.

If every plant knew about specific watering tools, adding another device could require changes across the plant hierarchy.

The project made me more aware that maintainability is often difficult to evaluate when a feature is first implemented.

A design becomes valuable when the **next requirement** arrives.

---

# 12. What I Would Do Differently Today

Looking back at the project, there are several areas I would improve.

## Environment-Based API Configuration

The external API configuration should be separated from source code.

I would prefer:

```text
Environment Variables
        ↓
Configuration
        ↓
API Client
```

This would provide a cleaner approach to credentials and environment-specific settings.

---

## Dedicated API Client

Instead of keeping HTTP concerns inside a utility-style class, I would likely define an explicit client abstraction.

For example:

```text
DialogueService
      ↓
LLMClient
      ↓
OpenAIClient
```

This would reduce coupling to one external provider.

It would also make it easier to replace the external API implementation.

---

## Structured Response Models

Generated Riddle responses should ideally be converted through structured DTOs.

For example:

```text
HTTP Response
     ↓
JSON Parser
     ↓
RiddleResponse DTO
     ↓
Domain Riddle
```

This would provide clearer validation and error handling.

---

## Better Failure Handling

External services can fail.

The game should therefore be able to continue even if the Oracle service is unavailable.

A stronger design would provide:

```text
External API Failure
       ↓
Fallback Dialogue
       ↓
Game Continues
```

The external integration should enhance the application rather than becoming a single point of failure.

---

## Dependency Injection

Some collaborators could be provided to classes rather than constructed internally.

For example:

```text
Oracle
  receives
DialogueGenerator / DialogueService
```

rather than always creating dependencies directly.

This could make components easier to test.

---

## More Automated Testing

Today I would place greater emphasis on automated tests for rule-heavy logic such as:

* behaviour selection;
* plant growth;
* dehydration;
* watering capacity;
* trading;
* API response parsing;
* riddle validation.

These systems contain deterministic business rules that are well suited to unit testing.

---

# 13. How This Project Influenced My Later Development

The biggest effect of this project was that I became more interested in **software structure**, not just whether a feature worked.

I started paying more attention to:

```text
responsibilities
dependencies
module boundaries
extensibility
maintainability
state transitions
```

When approaching later projects, I became more likely to ask:

> What should the architecture look like before I start adding everything into one file?

and:

> If another developer needs to change this feature later, what part of the system would they need to understand?

This project therefore helped move my thinking from:

```text
Implement Feature
```

toward:

```text
Understand Requirement
       ↓
Understand Existing System
       ↓
Choose Responsibility
       ↓
Define Boundary
       ↓
Implement Feature
       ↓
Evaluate Future Change
```

---

# 14. Design Patterns Became Tools Instead of Goals

One of my most important lessons was changing how I viewed design patterns.

Before:

```text
"I know Strategy Pattern."
"I know interfaces."
"I know inheritance."
```

After the project, the more useful questions became:

```text
What problem am I solving?

What is actually changing?

Do I need an abstraction here?

Will this make the next change easier?

Is this pattern reducing coupling or just adding more classes?
```

This distinction became important because it is possible to create code that contains many design patterns while still being difficult to maintain.

The goal is not:

> Use more patterns.

The goal is:

> Create a design that makes responsibilities and changes easier to manage.

---

# 15. Main Technical Takeaways

The project gave me practical experience with:

* extending an existing Java framework;
* understanding unfamiliar code before modifying it;
* object-oriented domain modelling;
* Action-based interaction design;
* Strategy Pattern;
* capability-oriented interfaces;
* inheritance and composition;
* stateful lifecycle modelling;
* separating domain and infrastructure concerns;
* external API integration;
* integrating AI output into application state;
* iterative software design.

More importantly, it changed how I evaluate code.

Instead of only asking:

> Does it work?

I began asking:

> Is the responsibility in the right place?

> What will happen when this requirement changes?

> Can another implementation replace this one?

> Does this class know more than it needs to know?

> Am I making the existing system easier or harder to extend?

---

# 16. Final Reflection

Elden Thing was an important transition point in how I approached software engineering.

It started as an object-oriented programming project, but the most valuable part was learning how multiple features interact inside an existing architecture.

The project showed me that software design is largely about managing change.

A system does not become maintainable because it uses inheritance, interfaces, or design patterns.

Those techniques are useful only when they help create clearer:

```text
Responsibilities
Dependencies
Boundaries
Extension Points
```

That is the main engineering lesson I took away from the project.

---

# Portfolio Notice

This reflection describes my technical understanding and learning from the project.

The original project was a team assessment built on a Monash University-provided game engine. The complete source code and assessment solution are intentionally not included in this public portfolio repository.

---

## Related Documents

* [Project Overview](../README.md)
* [Architecture Overview](architecture.md)
* [Design Decisions](design-decisions.md)
