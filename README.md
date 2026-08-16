# Elden Thing — Java OOP Portfolio

A Java object-oriented game project developed for **FIT2099 Object-Oriented Design and Implementation** at Monash University.

The original project was built on top of a **game engine provided by Monash University**. Our team extended the engine through its existing Actor, Action, Item, Ground, World and GameMap abstractions to implement custom game systems.

> This repository is a portfolio showcase only.
>
> The original source code and university-provided game engine are not included to respect academic integrity and distribution restrictions.

---

## Project Overview

Elden Thing is a turn-based console RPG inspired by Elden Ring.

The project extends an existing Java game framework with systems including:

- NPC behaviour and decision making
- combat
- farming and plant lifecycle
- watering systems
- trading and economy
- multiple maps and teleportation
- creatures and reproduction
- boss mechanics
- AI-powered NPC dialogue

The main engineering challenge was not building a game engine from scratch, but learning how to **understand and extend an existing object-oriented framework without breaking its architecture**.

---

## Tech Stack

- Java
- Object-Oriented Programming
- Strategy Pattern
- Polymorphism
- Inheritance and Interfaces
- Composition
- OpenAI API
- HTTP streaming
- UML / Software Design Documentation
- Git

---

# Architecture

The original project separates the university-provided engine from the game-specific implementation:

```text
Application
    │
    ▼
Monash FIT2099 Game Engine
    │
    ├── Actor
    ├── Action
    ├── Item
    ├── Ground
    ├── GameMap
    └── World
          │
          ▼
Game-Specific Extensions
    │
    ├── Actors
    ├── Actions
    ├── Behaviours
    ├── Items
    ├── Plants
    ├── Watering
    ├── Dialogue
    └── World Configuration
