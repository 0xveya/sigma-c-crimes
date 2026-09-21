# Sigma C Crimes

The front door to the completely reasonable C23 stack I am building to learn
C by rebuilding far too much of userspace.

This is also an experiment in how much dumb stuff I can make C do with macros,
`typeof`, `typeof_unqual`, `_Generic`, `static_assert`, `constexpr`, and whatever
other compile-time abuse C23 allows. I have a macro problem.

This repository is the ecosystem guide. The libraries remain small, separate,
and useful on their own.

## The vision

The goal is a typed userspace foundation that keeps C's directness while
abusing C23 hard enough to make stronger contracts visible at compile time.

- generated `Result(T, E)`, `Option(T)`, collections, and traits instead of
  untyped containers or sentinel-value error protocols
- explicit owning, borrowed, moved, cloned, and forgotten states, with scoped
  cleanup and useful moved-from diagnostics
- runtime-assisted RAII where owning types implement a `Drop` trait and
  `sigma_rt` makes sure it runs when their scope ends
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

## Where the crimes happen

The most cursed parts currently live in `sigma_libft`:

- [`meta.h`](https://github.com/0xveya/sigma_libft/blob/main/include/sigma/meta.h)
  is the macro engine: recursive expansion, argument counting, repetition, and
  the bounded 0-64 mapper that feeds the other crimes.
- [`printf.h`](https://github.com/0xveya/sigma_libft/blob/main/include/sigma/printf.h)
  is the cursed typed `printf`: `_Generic` turns values into borrowed format
  arguments, variadic macros map whole argument lists, and custom types dispatch
  through formatter vtables.
- [`type_registry.h`](https://github.com/0xveya/sigma_libft/blob/main/include/sigma/type_registry.h)
  is the central X-macro registry for character, owning, formatting, and custom
  types.
- [`traits.h`](https://github.com/0xveya/sigma_libft/blob/main/include/sigma/traits.h)
  builds clone, deinit, replace, and character traits from those registries with
  `_Generic` dispatch.
- [`ownership.h`](https://github.com/0xveya/sigma_libft/blob/main/include/sigma/ownership.h)
  implements checked move, take, pointer-move, swap, zero, and forget operations
  with `typeof` and compile-time type checks.
- [`vec.h`](https://github.com/0xveya/sigma_libft/blob/main/include/sigma/vec.h)
  generates borrowed, trivial, and owning vector types while preserving their
  different clone, move, and destruction contracts.
- [`diagnostic.h`](https://github.com/0xveya/sigma_libft/blob/main/include/sigma/diagnostic.h)
  makes `static_assert` emit structured Sigma error codes, the rejected source
  expression, and recovery help.
- [`sigma-diagnostics`](https://github.com/0xveya/sigma_libft/blob/main/tools/sigma-diagnostics/main.go)
  is the small Go compiler wrapper that parses those structured assertions and
  redraws them as readable source diagnostics while preserving the compiler's
  exit status.

Together, the type registry, traits, ownership rules, generated containers,
typed formatting, and compile-time diagnostics are the start of a tiny type
system built on top of C's type system. This is not because C needed another
type system. It is because I want to find out how far the macros will let me go.

The next ownership crime is runtime-assisted RAII. Owning types will register a
`Drop` trait, moves will disarm the old owner, and `sigma_rt` will help call the
right destructor automatically at scope boundaries instead of relying on every
return path to remember cleanup.

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
