---
id: task-25
title: 'codegen: concurrent tokio runtime for later/when/wait/ignore'
status: Done
assignee: []
created_date: '2026-09-29 11:05'
updated_date: '2026-09-29 20:42'
labels:
  - camus-pl
dependencies:
  - task-21
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Replace the synchronous prototype execution model (TASK-21, decision 3) with a real CONCURRENT runtime based on tokio: later spawns tasks, when branches on completion, wait awaits the handle, ignore detaches. Required before Camus PL is considered production-ready. Deferred by design from TASK-21; do not implement a concurrency runtime in the generated prelude until this is done.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 A tokio-based runtime crate is used for generated code (later spawns, wait awaits, when branches, ignore detaches)
- [x] #2 The sample-project CLI still works and behaves identically to the synchronous prototype
- [x] #3 Tests cover concurrent later/when/wait/ignore behavior
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Fossil: a32d5424a42d940c91d3b7924ab74ca666764823

## What was built

The executor lives in its own crate, `Camus-PL/runtime` (`camus-runtime`), which
generated code links. The generator emits no scheduling code of its own; the
prelude only re-exports `CamusResult`.

    later <call>  ->  camus_runtime::later(move || <call>)   -- dispatch, bind a handle
    when <handle> ->  match camus_runtime::wait(&h) { .. }  -- wait, then match
    wait <handle> ->  camus_runtime::wait(&h)               -- wait, discard the outcome
    ignore <call> ->  camus_runtime::ignore(move || {..})  -- dispatch and detach

### The handle is not a value

A `later` handle and a `Result` value are the same Camus type but different Rust
types, and that is what the lowering turns on:

    later <call>           ->  camus_runtime::Task<T>   (not completed)
    a call returning Result ->  CamusResult<T>          (already completed)

So `when` has two targets and lowers differently: over a handle it waits first,
over a value it matches in place. `match` is the immediate form and stays legal
only on a value. A handle caches its outcome, so `when h` followed by `wait h`
observes the same value twice.

### Generated code stays synchronous

The alternative was to make the whole call graph `async fn` and propagate
`.await` through every method, trait, law and the composition root. Rejected: it
would change the shape of code that has nothing to do with concurrency, and a
Camus call is ordinary blocking code (an LCR adapter does file or network I/O).
The runtime dispatches onto tokio's blocking pool, and `wait` blocks the calling
thread.

### Two consequences visible in the generated Rust

- LCR traits gain `Send + Sync` supertraits. Without them the `&'static
  dyn TaskStore` handed out by the composition root is neither `Send` nor
  `Sync`, and the closure is rejected.
- `self` cannot be captured (a `&self` receiver is not `'static`), so a
  dispatched call mentioning `self` hoists `let mut __camus_self =
  self.clone();` before the dispatch and the closure body is retargeted at it.

### `camus build`

The generated Cargo project declares `camus-runtime` as an absolute path
dependency (`--out` may place the project anywhere, so a relative `../runtime`
would not resolve).

## Tests

- `Camus-PL/runtime/tests/concurrency.rs` (9): non-blocking dispatch
  (deterministic, via a gate), work runs off the calling thread, `Err`
  propagation, repeated observation, concurrent waiters, detach, overlapping
  dispatches, panic containment.
- `codegen/tests/generate.rs`: the four constructs lower as expected; both
  dispatches precede both waits, which is what makes the concurrency real; the
  `__camus_self` alias does not leak.
- New fixture `codegen/tests/fixtures/asyncall/` exercises all four constructs
  including `ignore` over `self`.
- `codegen/tests/entrance.rs` / `build_cmd.rs`: shape assertions for the
  concurrent lowering and the emitted runtime dependency.
- All harnesses that drive `rustc` directly now build the runtime and pass
  `--extern camus_runtime` plus `-L dependency`.

checker 33 / codegen 19 / runtime 9 green; clippy clean on all three crates.

## Notes and follow-ups

- `ignore` gives no completion guarantee: the detached work races process exit.
  The `asyncall` fixture shows this, and it is the intended semantics.
- A panicking dispatch becomes an `Err` outcome rather than leaving `wait`
  without a result.
- Pre-existing, untouched: `is_result_value` stores `"Result"` in the
  translation environment but tests for a `"CamusResult"` prefix, so `return
  <result local>` inside a `Result` function re-wraps instead of forwarding. No
  fixture exercises it.
- Pre-existing, untouched: `sample-project` fails `camus-check` with 3 errors
  (a TCC imports an application LCR directly), unrelated to this task.
<!-- SECTION:NOTES:END -->
