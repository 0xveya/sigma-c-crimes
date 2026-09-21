---
name: sigma-c-setup
description: Set up a Sigma-style C23 repository with Xmake targets and packages, mise tasks, and Clang tooling. Use when starting or standardizing one of Veya's C repositories; not for a strict 42 hand-in unless the subject permits this tooling.
---

# Sigma C setup

Read the target and relevant sibling Sigma repositories first. Match their
current package declarations and keep only files the first target uses.

## Base

Use `src/`, `include/` for public library headers, and `tests/` when tests exist.
Add `xmake.lua`, `mise.toml`, `.gitignore`, and `.clang-format` containing
`Standard: Latest`. Ignore `.xmake/`, `build/`, `compile_commands.json`, object
files, produced binaries, swap files, and generated diagnostic files.

Start Xmake with:

```lua
set_project("<name>")
set_version("0.1.0")
set_languages("c23")
set_toolchains("clang")
set_toolset("ld", "clang")
add_rules("mode.debug", "mode.release")
add_rules("plugin.compile_commands.autoupdate", {outputdir = "."})
```

Factor shared target settings into `configure(target_name)`. Use warnings
`all`, `extra`, and `pedantic`, plus `-Wshadow`, `-Wconversion`,
`-Wdouble-promotion`, `-Wformat=2`, and `-Wundef`. Add freestanding flags only
for a real freestanding boundary.

Static libraries use public include directories, `add_headerfiles`, and
`add_files`. Test binaries are non-default, depend on the library, and register
focused cases with `add_tests` rather than adding another test framework.

Declare Sigma dependencies as GitHub-backed Xmake packages. Pin an existing
release with `add_requires(..., {system = false})`; copy install options from a
working consumer. Mark packages public only when their types appear in public
headers. Add only the needed layer: `sigma_malloc`, `sigma_libft`, or `sigma_rt`.

## Mise

Pin Clang, Xmake, and clang-format. Add other tools only when a task uses them.
Provide these tasks with short literal descriptions:

```toml
[tasks.build]
run = "xmake f -m release -y && xmake -y"

[tasks.dev]
run = "xmake f -m debug -y && xmake -y"

[tasks.test]
run = "xmake test -vD"

[tasks.compiledb]
run = "xmake project -k compile_commands ."

[tasks.clean]
run = "xmake clean -a && rm -rf build compile_commands.json"
```

Add `format` over the repository's actual C and header paths. Add `run`,
`check`, stress, diagnostics, or generation tasks only when they execute real
repository behavior.

## Verify

Run `mise run format`, `mise run dev`, and the narrowest relevant test. Run the
release build when its flags or linkage differ. Keep Xmake failures visible.
