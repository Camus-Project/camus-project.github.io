---
layout: page
title: Camus Specification Language
---

Camus SL is an **embeddable specification language** designed to make AI-generated code auditable, constrained, and certifiable by a human reviewer.

It is not a new programming language.
It is not a linter.
It is not a documentation tool.

It is a **semantic discipline layer** that lives inside existing programming languages through structured comments.

Its purpose is simple:

> If a human is to assume responsibility for code, that human must be able to understand exactly what the code claims to do.

Camus SL exists to make those claims explicit.

---

## Why Camus SL Exists

AI can generate working code.
Working code is not the same as trustworthy code.

Trust requires:

* Explicit intent
* Explicit terminology
* Explicit constraints
* Explicit human validation

Camus SL introduces a structured way to express:

* **What concepts exist in the project**
* **What each function claims to do**
* **What it requires to operate**
* **What it guarantees in return**

Without these elements, review becomes guesswork.
With them, review becomes an act of responsibility.

---

## What Camus SL Is

Camus SL is a **Specification Overlay**.

It is embedded inside host languages via structured comments and defines:

1. A **Lexicon** of project terms
2. A set of **function declarations** based on those terms
3. Explicit **claims** about behavior
4. Optional **constraints**
5. Controlled, documented **exceptions**
6. **Semantic contracts** between terms and implementations
7. **Signed function protection** against unauthorized modification

Camus SL does not execute.
It does not compile.
It constrains meaning.

---

## Core Concepts

### 1. The Lexicon

Every project defines a finite set of **Terms**.

A term describes *what something means* in the project context.
Not its data type.
Not its implementation.
Its meaning.

Terms are realized by components in the host language.

Without a lexicon, there is no shared vocabulary between human and AI.
With a lexicon, ambiguity shrinks.

---

### 2. Function Claims

Every function must declare:

* The **terms it expects**
* The **terms it returns**
* The **claim** it makes

The claim answers one question:

> What does this function assert it does?

This shifts code from "procedure" to "commitment."

---

### 3. Constraints

Functions may declare constraints on the terms they receive or produce.

Constraints must be:

* Explicit
* Checkable
* Intentional

No hidden assumptions.

---

### 4. Structured Exceptions

Camus SL is strict by design.

But real systems are messy.

Any deviation from rules must be:

* Explicitly declared
* Scoped
* Documented

Silence is forbidden.
Transparency is mandatory.

---

### 5. Semantic Contracts

Terms define **actions** — expected behaviors.

A function's SL declares:

* The actions it **expects** from its parameters
* The actions it **provides** in return

A term without actions is an empty concept.
A term with actions is a contract.

Semantic continuity follows this path:

> intent → action → term → implementation

This means:

* The **intent** of a function determines which **actions** it needs
* Each **action** is defined in the Lexicon as part of a **term**
* Each **term** is realized by a **component** in the host language
* The component's implementation must fulfill the actions it declares

Traits (interfaces) are compatible with Camus only if they carry explicit semantic contracts. A trait without semantic contract is prohibited.

---

## Grammar Rules

The following 15 rules define how code MUST be written under Camus SL.
Every rule may be excepted, but exceptions MUST be documented (see Exceptions below).

1. **Term Definition** — Every term MUST be defined in the project Lexicon
2. **Component Realization** — Each component MUST realize exactly one term, explicitly declared
3. **Function Terms** — Every function MUST declare its input and output terms
4. **Function Claim** — Every function MUST declare what it does (its claim)
5. **Function Constraints** — Declare explicit, verifiable constraints on parameters
6. **Block Depth** — Functions MUST limit block depth to one level (no nested blocks)
7. **Line Length** — Lines MUST not exceed 120 characters (warn above 80)
8. **Function Length** — Functions MUST not exceed 50 lines (warn above 20)
9. **Parameter Passing** — All parameters MUST be passed by reference
10. **Variable Mutability** — Variables MUST be immutable by default
11. **No Public Primitives** — Never expose primitives as public variables or return values
12. **No Anonymous Functions** — Every function must have an explicit name
13. **No Inheritance** — Favor composition over inheritance
14. **No Interfaces without Contracts** — Components communicate through explicit terms only; traits without semantic contracts are prohibited
15. **Explicit Exceptions** — Deviations are allowed only if explicitly documented with why, how, and risks

---

## What Camus SL Is Not

Camus SL is not:

* A domain-specific language
* A replacement for your host language
* A documentation generator
* A static type system
* A runtime verification framework

It is a **semantic contract framework** between:

* The AI that writes the code
* The human who audits and signs it

---

## The Responsibility Model

Camus SL is built around one idea:

> Software distribution implies responsibility.

A human reviewer may choose to certify a function.
But certification is meaningful only if:

* The function's intent is explicit
* Its dependencies are explicit
* Its constraints are explicit
* Its structure is limited and readable

Camus SL structures this process so that responsibility is not symbolic, but traceable.

---

## Signed Function Protection

Once a function is signed, the AI may not modify it without explicit developer approval.

No workarounds are permitted:

* Renaming a function to create a new version while abandoning the old one is prohibited
* Refactoring remains an objective to limit code drift
* The developer's signature is a commitment that must not be circumvented

If a signed function needs modification, the AI must:

1. Identify the function and its current signature
2. Explain why the modification is needed
3. Request explicit approval from the developer
4. Only proceed after approval is received

---

## Exceptions

Any Camus rule may be excepted.

Documentation is mandatory in the SL block:

* **Why**: the reason for the exception
* **How**: the implementation approach
* **Risks**: what the exception implies

Workflow:

1. AI identifies need for exception
2. AI requests human approval
3. Human approves or rejects
4. Only after approval: implementation

Presenting a fait accompli at review time is forbidden.

---

## Gradual Adoption

Camus SL is designed for incremental integration.

You may:

* Apply it to a subset of files
* Restrict it to critical modules
* Relax certain rules at project level

But every relaxation must be declared.

Camus does not punish partial adoption.
It measures clarity.

---

## Status

Camus SL is an early-stage project.
Its goal is to define the minimum semantic structure required for:

* Human auditability
* AI collaboration
* Explicit responsibility

The language, tooling, and conventions are evolving.

Discussion and critique are welcome.
