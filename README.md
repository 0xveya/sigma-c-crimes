# Sigma C Crimes

The front door to the completely reasonable C23 stack I am building because
apparently libc was not enough.

This repository is the ecosystem guide. The libraries remain small, separate,
and useful on their own.

## The vision

The goal is a typed userspace foundation that keeps C's directness while
abusing C23 hard enough to make stronger contracts visible at compile time.

- generated `Result(T, E)`, `Option(T)`, collections, and traits instead of
  untyped containers or sentinel-value error protocols
- explicit owning, borrowed, moved, cloned, and forgotten states, with scoped
  cleanup and useful moved-from diagnostics
- async/await, tasks, cancellation, channels, and structured concurrency in
  the runtime rather than a different event loop in every program
- libc-shaped breadth with bounded strings and bytes, typed I/O, files,
  processes, time, networking, Unicode, and platform services
- explicit allocator propagation and no hidden allocation
- typed syscall boundaries, useful compile-time diagnostics, and generated
  platform glue instead of handwritten repetitive wrappers
- hosted, freestanding, and 42 allowed-function builds from the same contracts
- Xmake packages that let each project compose only the layers it needs

The point is not to disguise C as another language. The point is to make
ownership, failure, allocation, and concurrency impossible to miss at the API
boundary while retaining predictable C data layouts and control flow.

## Library stack

- [sigma_malloc](https://github.com/0xveya/sigma-malloc): composable allocator
  interfaces, mmap and malloc memory sources, slabs, buddy allocation, arenas,
  and debug ownership diagnostics. It is in a fine state for experiments and
  the rest of this stack, but it is not production-ready. It still needs a lot
  of optimization, measurement, hardening, and general polish.
- [sigma_libft](https://github.com/0xveya/sigma_libft): the C23 foundation
  library. It currently provides bounded strings and bytes, owning strings,
  generated vectors, traits, ownership operations, formatting, Unicode,
  hash maps, readers, SIMD memory paths, and typed Linux syscall results.
  Generated `Result(T, E)` and `Option(T)` types belong here and are planned.
- [sigma_rt](https://github.com/0xveya/sigma_rt): the runtime layer for process
  startup, arguments, environment, TLS, allocator wiring, and eventually
  async/await and structured concurrency.

`sigma_libft` is the standard/foundation library, not the umbrella. New pieces
belong in the smallest layer that can stand alone instead of turning it into a
monolith.

```text
sigma-c-crimes
    |
    +--> sigma_malloc
    +--> sigma_libft
    |        +--> sigma_malloc
    +--> sigma_rt
    |        +--> sigma_libft
    |        +--> sigma_malloc
```

## Contract direction

- Borrow by default. Ownership transfer must be explicit in the function name
  or type contract.
- A moved-from value has one documented zero state and remains safe to destroy.
- Expected absence is an `Option`; reportable failure is a `Result`.
- Allocation is supplied by the caller or made explicit by the API name.
- Blocking work is visible. Async tasks live in a scope and cannot silently
  outlive it.
- Platform and libc calls cross one typed boundary so 42, hosted, and
  freestanding implementations can be swapped without changing callers.
- Type abuse should remove invalid programs or repetitive glue. It should not
  obscure runtime behavior.

Some of these contracts are already implemented and some are the roadmap. Each
library README is the source of truth for its current surface.

## Start a project

Use the included [sigma-c-setup skill](skills/sigma-c-setup/SKILL.md) when
asking an agent to start a project on this stack. It chooses only the layers the
project needs and preserves hosted, freestanding, and 42 constraints.
