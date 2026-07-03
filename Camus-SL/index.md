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
