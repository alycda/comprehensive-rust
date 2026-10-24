---
title: Unsafe Deep Dive, Two Full Days - Plan
type: feat
date: 2026-10-24
topic: unsafe-deep-dive-two-days
artifact_contract: ce-unified-plan/v1
product_contract_source: ce-brainstorm
execution: code
---

# Unsafe Deep Dive, Two Full Days - Plan

## Goal Capsule

- **Objective:** A facilitator can teach the Unsafe Deep Dive in this fork as two full classroom days, and a student who already has Rust fundamentals leaves able to call their own Rust from a language they have never written, because they learned to read the boundary rather than the language.
- **Means:** Fill the WIP deep dive in upstream's order. Day 1 is the existing foundations timed and completed. Day 2 is an FFI capstone ported from the facilitator's RustConf 2026 workshop and made generic.
- **Product authority:** The facilitator (repository owner). Upstream google/comprehensive-rust is not a stakeholder of this work.
- **Open blockers:** None. Everything unresolved is deferred to planning (see Outstanding Questions).

---

## Product Contract

### Summary

Complete the deep dive to its declared 600-minute target: time every Day 1 slide, write the title-only stubs, wire the six orphan pages, and build Day 2 as an FFI capstone that carries a small text-processing library across four boundaries in turn. Wrap C, export to C, call it from Python over the generated header, and export raw wasm32 functions driven from JavaScript. A React Native segment is written as timing buffer and kept or cut after the dry run.

### Problem Frame

The deep dive declares a 600-minute target in its welcome page, but the slides listed in the table of contents sum to 337 minutes, and 25 of the 77 listed slides carry no timing at all. Six pages are a licence comment and a title. Six written pages are not reachable from the table of contents.

The FFI segment covers one direction only, Rust calling C. Nothing shows Rust exposed to C, a callback handed to C, an error or a panic crossing the boundary, or a safe wrapper over an opaque handle, although the segment's own comparison tables name every one of those. Both FFI exercises end by asking learners to write the safe wrapper and give no model answer.

The facilitator has already built and delivered the missing half as a half-day RustConf workshop: a tested library of Advent of Code solutions crossed into C, Python and three other languages, with the failures that happened recorded as teaching material. That material is puzzle-specific and multi-language, and the course is neither. The cost today is that the facilitator cannot teach two days from this fork, and would have to teach the FFI half from a separate deck and repository with a different setup story.

### Key Decisions

- KD1. **Fork-only target; nothing is pushed.** (session-settled: user-directed — chosen over upstreaming to google/comprehensive-rust: upstreaming needs the CLA and Google headers, and the facilitator wants to teach first.) Governs R31.
- KD2. **The "what crossing costs" benchmark slide stays out.** (session-settled: user-directed — chosen over including it as an optional slide: the numbers are machine-specific and the strategies slide already gestures at cost.)
- KD3. **Two full days against the course's 600-minute target.** Half-day FFI-only and one-day formats were rejected because the facilitator will have the room for two days. Governs R1, R2, R18.
- KD4. **Capstone arc, upstream's order.** Day 1 foundations as upstream laid them out, Day 2 FFI, with two forward cross-link slides instead of a boundary-first reorder. Keeps upstream merges cheap and spends authoring where the facilitator has material. Governs R5.
- KD5. **The FFI segment is the RustConf workshop made generic.** C in both directions, Python over the C header, raw wasm32 from JavaScript. Kotlin, Swift, Dart, Fortran, R and Godot tracks are out; Python and JavaScript are the two most used consumer languages and React Native is more used than Flutter. Governs R6 to R17.
- KD6. **Raw wasm32 exports and linear memory, not wasm-bindgen or the component model.** The raw form has the same shape as the C glue and reinforces the unsafe lessons; the tooling slide names the alternatives. Governs R16, R17.
- KD7. **Python is cffi reading the generated header; PyO3 is named only.** Same header as the C harness, so the lesson stays "read the contract", not "learn a binding generator". Governs R15.
- KD8. **The running example is a handful of Advent of Code days chosen for byte-heavy input, used as is and made generic later.** Text crossing a boundary is where FFI is hardest, and kata-sized functions do not exercise it. The pick is Mull It Over (2024 day 3), Trebuchet (2023 day 1) and Historian Hysteria (2024 day 1). Governs R22, R23.
- KD9. **Consumer-language code is a scaffold with only the boundary left blank.** The outcome is writing a language you do not know, so students complete Python and JavaScript callers from the header, and solution pages gloss the foreign lines. Governs R20, R21, R24.
- KD10. **Non-FFI segments are timed and their stubs written; no new non-FFI segments.** The facilitator's material covers FFI only. Governs R2, R3, R4.
- KD11. **A tool-agnostic prework setup page with a self-check.** Chosen over Nix-first, install-during-class, and instructor-only toolchain: students install five tools before Day 1 and Nix is mentioned as one route. Governs R27 to R29.
- KD12. **Standard course headers on every new page; no provenance lines.** Chosen over per-page attribution to the facilitator or upstream-ready attribution notes. Governs R30.
- KD13. **React Native is a stretch segment, written and timed, kept only if the dry run leaves the total short of 600 minutes.** Governs R18, R19.
- KD14. **Source material is alycda/rustconf2026 on main.** The develop branch's Fortran, R and Godot tracks and the private aoc-ffi-workshop repository are not inputs.

<!-- ce-section: work-relationships -->
### How This Work Fits Together

This plan covers the whole Unsafe Deep Dive in this fork; the breakdown below is the current understanding, not a committed roadmap.

- Facilitator `.cheat` sheets for every course segment, including this deep dive.
  - Can proceed independently of this plan; it reads the finished slides, so it is cheapest after them.
- Making the running example generic (replacing the Advent of Code solvers).
  - Depends on this plan having been taught once; shares R22's publishing rule.
- An upstream contribution to google/comprehensive-rust.
  - Still to decide; depends on KD1 being revisited and on the attribution choice in KD12.

### Actors

- A1. Facilitator: the repository owner, who runs the two days from this fork and performs the dry run.
- A2. Student: has completed Rust Fundamentals or equivalent self-directed training, has not necessarily written Python or JavaScript.

### Requirements

**Course structure and timing**

- R1. The course outline for the Unsafe Deep Dive reports at least 600 minutes of content when the React Native segment is included; the course-structure page's outline directive is corrected to name the course as `welcome.md` declares it, since today it names a course that does not exist and renders nothing.
- R2. Every page listed under the Unsafe section of the table of contents carries a `minutes:` value.
- R3. The six written pages not currently reachable from the table of contents are reachable from it, each in the segment its content belongs to.
- R4. The six title-only pages carry slide content and speaker notes consistent with the segment around them.
- R5. The Initialization and Pinning segments each gain one short cross-link slide pointing forward to the Day 2 moment that depends on them.

**FFI segment content**

- R6. A slide wraps `qsort(3)` to teach a C function pointer type, a Rust-defined callback handed to C, and a slice crossing as pointer plus length plus element size, with the `a - b` comparator overflow as the live beat.
- R7. A slide shows Rust exposed to C: a dynamic library crate, an exported unmangled function, the four-step ingress for a C string, and a header generated from the Rust.
- R8. A slide teaches error reporting across the boundary as status codes and out-parameters, distinguishing input refused by the boundary from failure in the solver.
- R9. A slide teaches that a panic reaching an `extern "C"` frame aborts the process on Rust 1.81 and later (before that it was undefined behaviour), that `extern "C-unwind"` is the ABI that lets a panic cross, that `catch_unwind` is a backstop rather than a guard, and that checked arithmetic is the only overflow guard independent of build profile.
- R10. A slide gives the model answer the two existing exercises lack: an opaque handle wrapped in a type whose destructor frees it, with a hand-freed version shown first for code review.
- R11. A slide shows compiling C from a build script with the `cc` crate behind a feature gate, replacing the setup page's Bazel TODO.
- R12. A slide explains when a C shim is the binding: macro-only libraries, version-renamed symbols, and struct layouts that depend on how the library was built.
- R13. A slide covers ownership of a C library's state across threads: per-handle, process-global, and per-thread scratch, each with the Rust construct that makes it sound.
- R14. The type-safety stub becomes a slide on three ways an integer type is wrong at the boundary: wrong width, wrong domain, and no type tag at all.
- R15. A slide and exercise call the exported library from Python through `cffi` using the generated header as the declaration source.
- R16. A slide and exercise export the library as raw wasm32 functions and drive them from JavaScript under Node, passing text in and results out through linear memory with exported allocation functions.
- R17. A tooling slide names the generators by direction: bindgen, cbindgen, cxx and autocxx, UniFFI, PyO3, wasm-bindgen and the component model, and names JSI and TurboModules as the React Native boundary.
- R18. A React Native stretch segment shows the same C-shaped library reached from a React Native app, carries its own `minutes:` values, and is marked as cut-candidate in its speaker notes.
- R19. When the dry run shows the total at or above 600 minutes without the React Native segment, the segment is removed from the table of contents and kept in the tree.

**Exercises**

- R20. A C-glue exercise has students wrap a running-example solver in an `extern "C"` function, generate the header, and pass a C harness that tests a null pointer and invalid UTF-8 before the happy path.
- R21. The Python and JavaScript exercises provide a runnable scaffold in which only the boundary calls are left for the student to write.
- R22. Running-example code uses only the statement examples from the chosen Advent of Code days; no real puzzle input and no puzzle text appears in the repository.
- R23. At least one running-example path exposes a panic reachable only from a foreign caller, demonstrated in the panics slide.
- R24. Each solution page for a Python or JavaScript exercise explains every line a reader who knows only Rust would not recognise.
- R25. Exercise code that needs a local toolchain is excluded from the book's own test run and is built and tested by a separate check that the facilitator can run with one command.
- R26. Slides whose code needs only the C standard library remain runnable in the playground.

**Student setup**

- R27. A prework setup page lists the required tools: a Rust toolchain with the `wasm32-unknown-unknown` target, a C compiler and linker, `cbindgen`, Python 3 with `cffi`, and Node.
- R28. The setup page gives one self-check command that compiles and links a real C program and reports each tool's readiness.
- R29. The setup page gives install routes per operating system package manager and names Nix as one optional route.

**Repository hygiene**

- R30. New pages carry the course's standard licence header lines and no attribution or provenance lines.
- R31. The work stays on the fork and is not pushed to any remote until the facilitator says so.
- R32. The book builds, the book's code tests pass, the workspace tests pass, and the formatter reports no changes.

### Key Flows

- F1. Day 2 boundary chain
  - **Trigger:** Day 2 begins after the Pinning segment and its cross-link slide.
  - **Actors:** A1, A2
  - **Steps:** Wrap `abs`, `rand` and `qsort` from Rust. Write a safe wrapper over an opaque handle. Export the running example to C and read the generated header aloud. Call it from a C harness with hostile input first. Complete the Python scaffold from the header. Complete the JavaScript scaffold against the wasm32 build. Tooling slide, then React Native if kept.
  - **Outcome:** Every student has one library crossing four boundaries, two of them in languages they did not know that morning.
  - **Covered by:** R6 to R10, R15 to R18, R20, R21

- F2. Prework setup
  - **Trigger:** A student receives the course link before Day 1.
  - **Actors:** A2
  - **Steps:** Open the setup page, install the listed tools by the route for their system, run the self-check, repeat until every required row is green.
  - **Outcome:** Day 2 exercises build on the student's machine without class time spent on installers.
  - **Covered by:** R27 to R29

- F3. Dry run and cut decision
  - **Trigger:** The facilitator rehearses both days with the completed material.
  - **Actors:** A1
  - **Steps:** Teach from the fork, correct `minutes:` values from observed timing, read the outline total, apply R19.
  - **Outcome:** The table of contents matches what the two days hold.
  - **Covered by:** R1, R2, R18, R19

### Acceptance Examples

- AE1. Hostile input at the boundary
  - **Covers R20.**
  - **Given** the C harness and a student's exported function
  - **When** the harness passes a null pointer, then a byte string that is not UTF-8
  - **Then** each call returns the input-refused status and the process does not crash.

- AE2. Panic seen once on purpose
  - **Covers R9, R23.**
  - **Given** the exercise scaffold with its `todo!()` still in place
  - **When** the student runs the C harness before implementing anything
  - **Then** the process aborts with Rust's cannot-unwind message and no value is returned to C.

- AE3. Writing a language you do not know
  - **Covers R21, R24.**
  - **Given** a student who has never written Python and the Python scaffold
  - **When** they fill the declaration from the generated header and the one call
  - **Then** the scaffold runs and prints the statement example's answer, and the solution page explains the encoding line they had to add.

- AE4. React Native kept or cut
  - **Covers R18, R19.**
  - **Given** the dry run has corrected every `minutes:` value
  - **When** the outline total without the React Native segment is 600 or more
  - **Then** the segment leaves the table of contents and stays in the tree; otherwise it stays listed.

- AE5. Exercises do not rot
  - **Covers R25.**
  - **Given** a change to any Day 2 exercise crate
  - **When** the facilitator runs the separate check
  - **Then** the C harness, the Python scaffold solution and the JavaScript scaffold solution all build and pass, and the book's own test run is unaffected.

### Success Criteria

- The facilitator can run both days from `mdbook serve` on this fork without a second deck or repository.
- A student with no Python or JavaScript completes the Python and wasm exercises using only the header and the scaffold.
- The course outline total meets the 600-minute target after the dry run, with or without the React Native segment.
- R32 holds on the final tree.

### Scope Boundaries

**Deferred for later**

- Replacing the Advent of Code solvers with generic text-processing functions once the segment has been taught.
- Facilitator `.cheat` sheets for the deep dive; planned separately.
- New Initialization or Pinning slides beyond the stubs and cross-links.
- An upstream pull request to google/comprehensive-rust.
- A browser-hosted wasm demo; Node only in this work.

**Outside this work's identity**

- Kotlin, Swift, Dart, Fortran, R and Godot tracks.
- UniFFI, wasm-bindgen and component-model build mechanics beyond the tooling slide.
- Nix, devcontainer, `just` and Jujutsu infrastructure from the workshop repository.
- The benchmark slide (KD2).
- Real Advent of Code inputs or puzzle text.

### Dependencies and Assumptions

- The source code and prose come from alycda/rustconf2026 on main, which is MIT-licensed by the facilitator; relicensing under the course headers is the facilitator's choice (KD12).
- Students arrive with Rust fundamentals and a machine on which they can install the five tools.
- Minutes written for the 25 untimed slides and for all new slides are estimates until the dry run corrects them.
- The course tooling computes session totals from per-slide `minutes:` frontmatter and renders them through the outline macro. The directive in `src/running-the-course/course-structure.md` currently asks for a course named `Unsafe` while `welcome.md` declares `Unsafe Deep Dive`, so the total renders as not found until R1's correction lands; after that, R1 and R2 are checkable from the built book.
- The playground links the C standard library, so `abs`, `rand` and `qsort` slides can stay runnable there.
- Advent of Code's request that inputs and puzzle text not be republished holds for this repository.

### Outstanding Questions

**Deferred to Planning**

- How the separate exercise check (R25) is wired: a workspace member built by the existing xtask test task, a dedicated xtask subcommand, or a script; and whether the deep dive's empty `Cargo.toml` becomes that crate.
- Which React Native mechanism the stretch segment shows, given the constraint that it reaches the same C-shaped library: JSI directly, a TurboModule, or a generated binding.
- The minimum Node version and whether the wasm exercise also targets `wasm32-wasip1`.
- The exact set of pages each orphan is attached under, and the minute estimates per slide.
- How the Python and JavaScript scaffolds are laid out so the book's own test run skips them while the separate check builds them.

### Sources and Research

- Deep dive state: `src/unsafe-deep-dive/welcome.md` (target), `src/SUMMARY.md` Unsafe section, title-only pages `src/unsafe-deep-dive/ffi.md`, `ffi/type-safety.md`, `initialization.md`, `safety-preconditions/semantic-preconditions.md`, `safety-preconditions/u8-to-bool.md`, `rules-of-the-game/soundness-proof.md`; orphan pages `motivations.md`, `introduction/defining-unsafe-rust.md`, `pinning/README.md`, `case-studies.md`, `case-studies/intrusive-linked-list.md`, `ffi/README.md`.
- Existing FFI slides whose tables name the gaps: `src/unsafe-deep-dive/ffi/language-differences/rust-and-c.md`, `rust-and-cpp.md`, `cpp-and-c.md`; existing exercises `ffi/c-library-example.md`, `ffi/cpp-library-example.md`; setup TODO in `src/unsafe-deep-dive/setup.md`.
- Course conventions: `STYLE.md`, `CONTRIBUTING.md`, `GEMINI.md`; test gates in `xtask/src/main.rs`; timing in `mdbook-course/src/course.rs`.
- Existing pattern for a workspace crate that compiles C++: `third_party/cxx/blobstore`.
- Workshop sources in alycda/rustconf2026 (main): `days/2024-12-01/src/qsort.rs`, `days/2024-12-01/src/c_api.rs`, `days/2024-12-01/cbindgen.toml`, `days/2024-12-01/build.rs`, `days/2024-12-01/src/uthash_wrapper.c`, `days/2015-12-01/src/tcc.rs`, `days/2015-12-05/src/icu_shim.c`, `days/2015-12-05/src/hyperscan.rs`, `days/2023-12-01/yara_shim.c`, `days/2023-12-01/src/espeak.rs`, `days/2021-12-02/src/duckdb.rs`, `days/2021-12-02/src/chipmunk.rs`, `exercises/ex2-c-glue/`, `.github/ci/exercises/ex2-c-glue/`, `.github/ci/exercises/ex3-bindings/python/bindings.py`, `book/src/boundary.md`, `scripts/self-check.sh`, `days/2024-12-03/src/c_api.rs`.
- Fundamentals exercises considered and set aside for the running example: `src/types-and-values/exercise.rs`, `src/testing/exercise.rs`.
