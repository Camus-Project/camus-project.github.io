# The Camus Project

> Misnaming things adds to the world’s suffering.<br>
> _Mal nommer un objet, c’est ajouter au malheur de ce monde._<br>
> — Albert Camus, *Sur une philosophie de l’expression*, 1944.

![License: MIT](https://img.shields.io/badge/License-MIT-green)
![Status: Concept](https://img.shields.io/badge/Status-Early%20Concept-blue)

## The AI Challenge

Artificial intelligence is revolutionizing the Art of Code. Beyond obvious benefits, new challenges are emerging. The most pressing is _trust_. The Camus project aims to address it by providing the required tools.

### The Problem of Trust

Ultimately, trust is a human problem, not a technical one. And no human problem has a purely technical solution, but technology helps to find a human one.

Trust rests on a promise made by humans. This promise binds them. This commitment is proven by signatures. Only then the technical aspect begins: it aims to guarantee the authenticity and persistence of these signatures. Because if the commitment is not honored, these signatures are used to demand redress, and this redress is refused if the signatures have not persisted until then or if their authenticity is not verified.

AI cannot sign. Or only by delegation from humans, who assume responsibility for the signature and any potential redress. AI is not a subject of law. Whether it could one day become so is more than hypothetical and debatable, and falls outside the scope of the problems that the Camus project aims to solve.

Moreover, if AIs were to become responsible, they would have every interest in using our tools to supervise their irresponsible agents and prove their good faith.

### The Solution, Ultimately

Distributing software entails responsibility. This responsibility is only assumed if the code is _mastered_.

The Camus project aims to produce the tools to measure this responsibility almost line by line. It involves _conceptualizing_ an entire project in the same terms as AI, _understanding_ the intent that each function claims to fulfill, and _certifying_ that the code it uses to do so actually achieves it.

At the end of this process, each function is explicitly **signed by a human auditor**, who assumes responsibility for it. 
Only then can the software be considered fit for distribution.

## The Camus Method

So we need to master AI code. We must conceptualize with the AI, understand its code, and certify it. What tools are required to enable this ?

### Requirements

Camus requires some structural elements from the host programming language in order to operate:

1. The ability to write comments, to host the Camus SL overlay.
2. The ability to define _components_, such as classes, structures, or objects.
3. The ability to define functions.

Most modern languages provide these three features.

### Conceptualizing — the _Lexicon_

The lexicon is the first requirement of the Camus Method.

A lexicon is a finite, explicit set of _terms_ describing _what_ the software manipulates.
Each term names a concept shared and understood by humans and AI.
Terms are _realized by_ components.

Terms may include subterms, which are themselves terms.
Each subterm is realized as a property of the realization of the term that contains it.

### Understanding — the _Grammar_

Grammar is a set of rules that the programmer, assumed to be an AI, MUST follow. These rules guarantee that the code is readable and understandable to a human reviewer.

It ensures that manipulations of terms are explicit, auditable, and predictable.

#### `kiss` as the Grammar Gardian

Ensuring respect for Grammar is the primary role of `kiss`, Camus' companion tool.

`kiss check` must be run before human review **and** compilation.

Its job is to verify that the programmer respects the Grammar rules, and that any exception to a rule is already **known, documented and intentional**.

#### The Rules

##### Rule 1 — Term Definition

Every term MUST be defined in the project lexicon.

##### Rule 2 — Component Realization

Each component MUST realize exactly one term, which MUST be explicitly declared.

##### Rule 3 — Function Terms

Every function MUST declare the terms it expects as parameters and the terms it returns.

##### Rule 4 — Function Claim

Every function MUST declare its claim.

##### Rule 5 — Function Constraints

A function MAY declare constraints on the terms it receives and returns. These constraints MUST be explicit and verifiable.


### Certifying — the _Signature_
