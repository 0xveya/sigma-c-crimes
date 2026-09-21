---
name: sigma-c-setup
description: Create a C23 project that uses the Sigma C ecosystem with Xmake, mise, explicit ownership and allocator contracts, and the correct hosted, freestanding, or 42 constraints. Use when starting or scaffolding a new Sigma C project.
---

# Sigma C setup

## Establish the boundary

Before creating files, inspect the target repository and relevant sibling Sigma
libraries. Determine:

- whether the output is a library or executable
- whether it is hosted, freestanding, or restricted by a 42 subject
- the target platforms and toolchain
- the allowed functions when a subject constrains them
- which Sigma layers provide contracts the project actually needs

Prefer the repository's existing conventions. For a new project, use C23,
Clang, Xmake, and mise unless its environment requires something else.

## Select the stack

Add only concrete dependencies:

- `sigma_malloc` for allocator interfaces, arenas, allocation strategies, or
  allocation diagnostics
- `sigma_libft` for bounded values, ownership helpers, generated collections,
  traits, formatting, Unicode, readers, memory operations, or typed syscalls
- `sigma_rt` for startup and runtime services

Treat generic `Result` and `Option` in `sigma_libft`, and async/await in
`sigma_rt`, as roadmap items until those repositories expose working public
contracts. Do not copy speculative versions into each new project.

When a package is not published through a registry, follow the dependency
pattern already used by the relevant Sigma repository. Do not invent a package
URL, version, or installation command.

## Preserve the contracts

- Use borrowed views by default and make ownership transfer explicit.
- Give every owning value a documented zero state that is safe to destroy.
- Use `SIGMA_MOVE`, clone, deinit, and allocator-aware APIs according to the
  owning type's contract. Ordinary C assignment is not a move.
- Pass allocators into code that allocates. Do not hide allocation in a helper
  with a borrowed-looking API.
- Represent expected absence and reportable failure with the strongest typed
  contract currently available. Do not collapse either into an undocumented
  sentinel.
- Keep libc and platform access behind one narrow boundary so hosted,
  freestanding, and allowed-functions implementations remain replaceable.
- Keep blocking operations visible. Do not create a project-local async macro
  system while the shared runtime contract is unfinished.
- Use metaprogramming where it enforces a contract or generates repetitive
  type-safe code, not merely to shorten ordinary control flow.

## Project shape

Keep the project as small as its deliverable permits. A typical new library has
public headers under `include/`, implementation under `src/`, focused behavior
tests under `tests/`, `xmake.lua`, and `mise.toml`. Do not create unused
directories or placeholder subsystems.

Expose one obvious mise task for each operation the project actually supports,
such as formatting, a debug build, a release build, and focused tests. Keep the
underlying Xmake failure visible.

## Verify

Run the narrowest checks that exercise the created project through its real
toolchain. At minimum, format the touched C and headers, build the requested
target, and run its focused behavior test when one exists. For restricted
projects, also verify the real allowed-functions or freestanding configuration;
a normal hosted build is not a substitute.
