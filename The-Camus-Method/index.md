---
layout: page
title: The Camus Method
---

The Camus Method is a methodology for writing, auditing, and certifying software using a hybrid human-AI process.
It is **language-agnostic** and can be applied on top of existing languages through embedded Camus annotations.

---

## Core Principles

1. **Human-in-the-loop certification**
   Code is written or suggested by AI and **audited, reviewed, and certified by a human**.
   The human is ultimately responsible for correctness, security, and semantics.

2. **Lexicon-driven semantics**
   Every project maintains a **lexicon of terms** that explicitly define the meaning of domain concepts.
   Functions, structures, and modules use these terms to ensure that the code is **self-descriptive and auditable**.

3. **Separation of intention and implementation**
   - **Terms** describe *what* a value or object means in the project context.
   - **Constraints** describe *what* a function requires to operate.

4. **Semantic constraints on composition**
   - Only terms can pass between functions.
   - Primitives can be used internally but cannot leak outside a function.

5. **Gradual coverage & auditability**
   - Projects can adopt Camus partially.
   - Coverage is measured in multiple dimensions: global, critical, strictness, and human-certified percentage.
   - Transparent reporting is mandatory for claiming "uses Camus."

6. **Complexity limitation**
   The methodology enforces constraints that limit cognitive and algorithmic complexity, ensuring maintainable, auditable code.

7. **Embedded in comments for external languages**
   - Camus annotations can be embedded in comments for any target language.
   - These annotations reproduce the **semantic contracts, term definitions, and constraints** of Camus.

---

## Workflow

1. **Lexicon proposal** – AI proposes a lexicon of terms for the domain.
2. **Lexicon validation** – Human reviews and certifies terms.
3. **Function skeletons** – AI proposes function signatures with intended terms.
4. **Human validation** – Human approves or adjusts skeletons.
5. **Implementation** – AI generates structures, interfaces, and function bodies.
6. **Testing & certification** – Human audits, tests, and certifies the code.

---

## Coverage and Reporting

The `kiss` tool (or equivalent) measures:

- **Camus Coverage:** proportion of code under Camus methodology
- **Critical Coverage:** coverage of project-critical areas
- **Certification Percentage:** proportion reviewed and approved by humans
- **Strictness Level:** rules enforced per module or function

Only with a **transparent report** can a project claim "Uses Camus."

---

## Philosophy

The Camus Method is **not a policing mechanism**.
It is a **trust and audit framework**:

- Emphasizes **responsibility over enforcement**.
- Promotes **gradual adoption**, highlighting progress rather than penalizing incompleteness.
- Transforms software development into a **measurable, semantically auditable activity**, independent of language.

> Camus does not certify code.
> Camus certifies the **degree of trust we can place in code**.

---

## Status

This project is a **foundational reference**, not a finished methodology.
It defines the **minimum rules** for code to be considered compliant with Camus principles and provides the basis for embedding Camus in any programming language.
