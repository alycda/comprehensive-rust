# Filling out the Unsafe Deep Dive from alycda/rustconf2026 and alycda/aoc-ffi

Status: proposal. Nothing changed in the repo.

## 1. Where the deep dive actually is

- `welcome.md` declares `target_minutes: 600` (two days). The `minutes:` frontmatter
  across all 83 slides sums to 343. Gap: ~257 minutes.
- Stub pages (title only): `ffi.md`, `ffi/type-safety.md`, `initialization.md`.
- Written but not in `SUMMARY.md`: `motivations.md`, `ffi/README.md`,
  `case-studies.md`, `case-studies/intrusive-linked-list.md`,
  `introduction/defining-unsafe-rust.md`, `pinning/README.md`.
- `setup.md` carries a TODO: "We should be able to avoid this [Bazel] by just
  relying on the `cc` crate". Nothing in the course uses `cc` yet.
- FFI section as shipped: language interop, strategies, five language-difference
  slides, `abs(3)`, `srand/rand(3)`, a C text-analysis library exercise (raw
  bindings given, "safe wrapper" left to learners with no model answer), a C++
  interner exercise (same shape). `ffi/README.md` says the section should
  "progress into more complex cases which involve pointers and uninitialized
  memory"; that progression is not written.

What is missing is one whole direction and several boundary concerns:

| Gap | Where the course mentions it | Where it is taught |
|---|---|---|
| Rust **exporting** to C (`cdylib`, `#[unsafe(no_mangle)]`, `cbindgen`) | nowhere | nowhere |
| Callbacks: fn pointer + `void*` userdata | `rust-and-c.md` table row | nowhere |
| Errors across the boundary: status codes, out-params | `rust-and-c.md` table row | nowhere |
| Panics across `extern "C"` | `rust-and-c.md`, `rust-and-cpp.md` rows | nowhere |
| Safe RAII wrapper over an opaque handle | both exercises ask for it | no model answer |
| Compiling C from `build.rs` (`cc` crate) | `setup.md` TODO | nowhere |
| Macro / header-only / version-renamed C APIs | nowhere | nowhere |
| Thread safety of C library state | `semantics.md` (ctime shared buffer) | one sentence |
| Type width traps beyond `rand() -> char` | `type-safety.md` stub | nowhere |

Every row has a worked, tested counterpart in your two repos.

## 2. The mapping

Source paths are under `alycda/rustconf2026` unless marked `aoc-ffi`.

### A. Callbacks: wrapping `qsort(3)` — `ffi/qsort.md` (15 min)

Source: `days/2024-12-01/src/qsort.rs` (also `aoc-ffi/aoc-2024-12-01/src/lib.rs`).

Slots directly after `abs.md` and `rand.md`: same libc-only shape, no build step,
so it runs anywhere the other two do. It adds the three things those slides do
not have: a C function pointer type in an `extern` block, an
`unsafe extern "C" fn` defined in Rust and handed to C, and a slice crossing as
pointer + length + element size.

Live-coding beat with a real history: write the comparator as `a - b`, ask the
class what is wrong, then show `i32::MAX` vs `i32::MIN` (panic in debug, wrap in
release). This is the bug that shipped live in your talk, and it links back to
the deep dive's own `introduction/may_overflow.md`. Your repo's agreement test
pins exactly those extremes; port it as the hidden test.

### B. Type safety across the boundary — fills `ffi/type-safety.md` (10 min)

Sources: `days/2021-12-02/src/duckdb.rs` (HUGEINT read through a raw `i64` cast),
`days/2021-12-02/src/chipmunk.rs` (f64 result checked for integrality and
range), `rand.md`'s existing `char` trick.

The stub needs content, and `rand.md` already makes the small point ("Rust will
trust you if you use the wrong type"). The DuckDB case makes the large one: a
columnar FFI read is a raw data pointer you `cast::<i64>()`, and if the column
is really 128-bit you read the low half and get a plausible answer with no
error anywhere. Fix belongs in the query (`::BIGINT`), not in Rust. One slide,
three rows: wrong width (rand/char), wrong domain (Chipmunk f64 for an integer
answer), no type tag at all (DuckDB). Keep DuckDB and Chipmunk as excerpts, not
runnable code; they need system libraries.

### C. Opaque handles and the safe wrapper — `ffi/safe-wrapper.md` (15 min)

Sources: `days/2015-12-01/src/tcc.rs`, `days/2015-12-01/src/caca.rs`, your
`HashEntry { _opaque: [u8; 0] }` pattern in `uthash.rs`.

The course's two exercises end at "now ask learners to create a safe wrapper"
with no worked example. `tcc.rs` is written deliberately without RAII: every
error path after `tcc_new()` calls `tcc_delete()` by hand, and the comment says
"no `?` between allocation and free". That makes it a code-review exercise in
the deep dive's own format: show it, ask the class what a `Drop` impl changes,
then show the RAII version (struct holding `NonNull<TCCState>`, `Drop` calls
`tcc_delete`, methods take `&mut self`). Place it before the C library
exercise so learners have the pattern when they reach it.

`caca.rs` is the smaller sibling if tcc is too much (libtcc is exotic). Either
is excerpt-only; neither builds in the playground.

### D. Compiling C from `build.rs` — `ffi/build-rs.md` (10 min)

Sources: `days/2024-12-01/build.rs`, `days/2024-12-01/Cargo.toml` (features,
`[build-dependencies] cc`), `aoc-ffi/aoc-2024-12-01/build.rs`.

Resolves the `setup.md` TODO in the direction its author wanted. Content: `cc::Build`,
`rerun-if-changed`, why `cc` is a plain build-dependency rather than optional
(build scripts see features as env vars, not `cfg`), feature-gating exotic
libraries so a bare `cargo test` stays green on a stock runner, and the `-O2`
you had to force for hardened glibc's `_FORTIFY_SOURCE` warning.

### E. When the API is a macro: shims — `ffi/shims.md` (10 min)

Sources: `days/2024-12-01/src/uthash_wrapper.{c,h}` + `uthash.rs`,
`days/2015-12-05/src/icu_shim.c`, `days/2023-12-01/yara_shim.c`.

Three reasons a C shim is the binding, not a workaround, each with a file:
uthash has no symbols at all until a translation unit instantiates its macros;
ICU renames every C symbol with a version suffix at link time, so a hand-written
declaration hardcodes today's version; YARA's match iteration is a macro walking
structs whose layout depends on how the library was configured. uthash is the
teaching example (three functions, fits a slide); ICU and YARA are one bullet
each with the rule "enumerate a library by what is exported, not what is
documented".

### F. Who owns the library's state — `ffi/global-state.md` (10 min)

Sources: `days/2023-12-01/src/espeak.rs` (static output buffer, process-wide
`Mutex`), `days/2015-12-05/src/hyperscan.rs` (shared database, per-thread
scratch via `thread_local!`), `days/2015-12-01/src/tcc.rs` (`static JIT: Mutex<()>`
guarding libtcc's lazily initialised semaphore), `days/2023-12-01/src/yara.rs`
(fresh `YR_RULES` per solve, no lock needed).

`semantics.md` already shows `ctime`'s shared buffer and says lifetimes cannot
express it. This slide answers the next question: so what do you write? Three
answers for three shapes of state (per-handle, process-global, per-thread
scratch), and the detail that `cargo test`'s default parallelism is what found
two of them. Natural home for the course's only other `unsafe impl Send/Sync`
discussion, which today lives in the tokio case study.

### G. The other direction: exporting Rust — `ffi/exporting.md` (15 min)

Sources: `days/2024-12-01/src/c_api.rs`, `days/2024-12-01/cbindgen.toml`,
`exercises/ex2-c-glue/Cargo.toml` (`crate-type = ["cdylib"]`).

Everything in the FFI section today is Rust calling C. This slide is Rust
exposing itself: `crate-type = ["cdylib"]`, `#[unsafe(no_mangle)] pub unsafe extern "C" fn`
and why edition 2024 requires the `unsafe` on the attribute (a symbol collision
is UB the linker arranges), the four-step ingress for a `*const c_char` (null
check, `CStr::from_ptr`, `to_str`, hand over), and `cbindgen` producing the
header. Your 2015-12-01 learning about aiming cbindgen at `src/c_api.rs` rather
than the crate root (otherwise it redeclares libtcc's imports in your header)
is a speaker-note beat.

### H. Errors across the boundary — `ffi/errors.md` (10 min)

Sources: `book/src/boundary.md` ("Errors: C has no `Result`"),
`days/2024-12-01/src/c_api.rs` (`-1` input refused, `-2` overflow, value through
an out-parameter).

Two kinds of status code with different owners: the boundary refused the input,
or the solver failed on valid input. Never a sentinel folded into the answer,
because any sentinel is a value some input can produce. And the honest part: a
status code does not force anyone to read it. Pairs with the existing
`rust-and-c.md` "Errors" row.

### I. Panics and the boundary — `ffi/panics.md` (15 min)

Sources: `days/2021-12-02/src/c_api.rs` module doc and `README.md` learnings,
`days/2023-12-01/README.md` ("Exposing a day to C found a panic the day could
not reach on its own"), `exercises/ex2-c-glue/README.md` step 0.

This is the strongest material you have, and the course has nothing on it:

- Rust ≥ 1.81 aborts when a panic reaches an `extern "C"` frame. Step 0 of
  your Ex 2 ("run the harness before implementing, watch
  `panic in a function that cannot unwind`") is a ready-made demo.
- `catch_unwind` as a backstop is not a guard: integer overflow panics only
  where `overflow-checks` is on, which is dev and not release. The first
  version of 2021-12-02's `-3` worked in debug and returned `0` plus a wrapped
  number in the build most likely to ship. `checked_*` arithmetic is the only
  version that does not depend on the profile. Also: `panic = "abort"` kills
  the process before `catch_unwind` runs, and the default panic hook prints
  to the host's stderr first.
- Valid UTF-8 and panic-free are different promises: 2023-12-01 sliced by byte
  offset, every Rust caller fed it ASCII, the suite was green, and `é1` from a
  C caller panicked inside the `extern "C"` frame.

### J. Exercise: C glue — `ffi/exercise-c-glue.md` + solution (30 + 15 min)

Source: `exercises/ex2-c-glue/` scaffold, `.github/ci/exercises/ex2-c-glue/`
solved overlay, `tests/c/test_glue.c`, `build-and-test.sh`.

A complete exercise with a four-step TODO, a C harness that tests the hostile
inputs (NULL, invalid UTF-8) before the happy path, and a 2025-day-3 solver
whose part-2 answer exceeds 32 bits on purpose so a binding that truncates to
`int` fails rather than passes by luck. Replace the Advent of Code solver with
something puzzle-neutral if you prefer (the course avoids external puzzle
dependencies), but the shape ports as is. Needs a local toolchain and a C
compiler, like the concurrency and bare-metal exercises; mark the blocks
`ignore` and say so on the slide, as `rand.md` already does.

### K. Tooling: generated bindings — `ffi/tooling.md` (5 min)

Sources: `aoc-ffi` (UniFFI: `uniffi::export`, `.udl`, generated Kotlin names
`AocError` → `AocException`), `days/2024-12-01/README.md` (autocxx emitted the
same `extern "C"` shim the port hand-writes).

`rand.md` ends with "avoiding type safety issues is a reason for using tools
for generating wrappers". One slide names them by direction: bindgen (C → Rust),
cbindgen (Rust → C), cxx/autocxx (C++), UniFFI (Rust → Python/Kotlin/Swift). Your
lesson "generated code is the source of truth, not your assumptions" is the
speaker note; the "one header, four treaties" table from `boundary.md`
(compiler-checked vs consumed vs transcribed vs remembered) is a strong optional
addition if you want the consumer side.

### L. Optional: what crossing costs — `ffi/cost.md` (5 min)

Sources: benchmark tables in `days/2024-12-01/README.md`, `days/2015-12-01/README.md`,
`days/2021-12-02/README.md`.

"The boundary's cost depends on how often you cross it": qsort pays an indirect
call per comparison (3.8× slower), uthash crosses twice in bulk and wins
(9.4× faster than the naive scan). Numbers are from your aarch64 machine; present
them as yours, not as general figures. Low priority; `strategies.md` already
gestures at zero-cost.

## 3. Proposed SUMMARY.md order for the FFI segment

```
- [FFI](unsafe-deep-dive/ffi.md)                          ← fill the stub from ffi/README.md
  - Language Interop, Strategies                           (as is)
  - Language differences + 5 children                      (as is)
  - abs(3), srand/rand(3)                                  (as is)
  - Wrapping qsort(3): callbacks                           A  new
  - Consideration: Type Safety                             B  fills stub
  - Opaque handles and safe wrappers                       C  new
  - Exercise: C library, Exercise: C++ library             (as is; now have a model)
  - Compiling C from build.rs                              D  new
  - When the API is a macro: shims                         E  new
  - Who owns the library's state                           F  new
  - Exporting Rust: cdylib, no_mangle, cbindgen            G  new
  - Errors across the boundary                             H  new
  - Panics and the boundary                                I  new
  - Exercise: C glue / Solution                            J  new
  - Tooling                                                K  new
  - (What crossing costs)                                  L  optional
```

New minutes: A15 B10 C15 D10 E10 F10 G15 H10 I15 J45 K5 = 160, L adds 5. Plus
wiring `motivations.md` and `case-studies.md` into SUMMARY (already written,
~10 min). That closes roughly two thirds of the 257-minute gap and finishes the
FFI segment's stated arc. The remaining gap is in initialization/pinning
("uninitialized memory" cases the FFI README promises), which your repos do not
cover.

## 4. What I would not port

- The language-track harnesses (Python cffi, Dart, Kotlin JNA, Swift). The deep
  dive is Rust ↔ C/C++; the Android deep dive already owns Java interop. Keep
  them as one table in K at most.
- Nix, devcontainer, `just` and jj material. Course setup is `cargo` only.
- Advent of Code inputs or puzzle text (your own rule, and AoC's).
- UniFFI build mechanics (`.udl`, scaffolding in build.rs). One mention in K.

## 5. Constraints to settle before writing slides

1. **Licensing.** `rustconf2026` is MIT (you, 2026). `aoc-ffi` has no LICENSE
   file. The course is CC-BY-4.0 prose and Apache-2.0 code, both Google LLC in
   the headers. In your fork you can relicense your own work however you like;
   to upstream to google/comprehensive-rust you would sign Google's CLA and the
   contributed text takes the course's headers. Decide which target you are
   writing for before I draft, because it changes attribution lines.
2. **Runnable vs excerpt.** A, B(rand part), G ingress code and I can run in the
   playground (libc links by default). C through F and J need a C compiler and
   system libraries: excerpts marked `ignore`, with the uthash wrapper as the
   one small C file that could ship in the repo under `src/unsafe-deep-dive/`
   and be built by `cargo xtask rust-tests` the way `third_party/cxx/blobstore`
   is today.
3. **Puzzle-neutral exercise or not.** J is strongest with your real 2025-day-3
   solver (the >32-bit answer is the point). The course avoids external puzzle
   dependencies; a synthetic function with the same property works too.
4. **Style.** `STYLE.md` and `GEMINI.md` govern: one H1 per slide, no H2 on
   slides, `minutes:` frontmatter, speaker notes in `<details>`, Apache header
   as `# //` hidden lines in every code block, `dprint fmt` after. Your
   READMEs' voice is denser than the course's bullet style; expect a rewrite,
   not a paste.

## 6. Side note on the `.cheat` proposal

`rustconf2026/.cheat/` already has your conventions: `demo/<beat>` sheets
numbered `1/4`..`4/4` with `syntax: bash` and `tags: [ demo ]`, a `demo/all`
sheet with the whole sequence, and `demo/clip` using `scripts/clip.sh` (OSC 52)
for the clipboard. I will mirror that in the comprehensive-rust sheets: live
sheets become `demo/<slide>` with numbered beats, and the root `course` sheet
documents the OSC 52 route rather than assuming `pbcopy`/`wl-copy`.
