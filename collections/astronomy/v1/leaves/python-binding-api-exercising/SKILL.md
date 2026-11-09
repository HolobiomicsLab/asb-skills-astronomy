---
name: python-binding-api-exercising
description: Use when a task requires verifying documented semantics of a library
  object model (e.g. FrameSet as Mapping vs as Frame) by driving its Python binding
  with scripted mutations and recording pass/fail behaviour.
when_to_use_negative:
- You need to build or install the library itself rather than exercise its API semantics
  — use the build/bootstrap workflow instead.
- The question concerns non-semantic properties (build configuration, compiler flags,
  F77/FC/CC environment) rather than object behaviour through the binding.
- The target library has no language bindings for the runtime you intend to drive
  (AST is written in C; its bindings cover Python, Java, Perl and Fortran).
tools:
- name: Starlink AST
  role: Object model under test — FrameSet and Frame objects whose Mapping semantics
    and integrity restoration are exercised
  repo: Starlink/starlink
- name: Python
  role: Language binding used to instantiate FrameSet/Frame objects and script the
    mutation checks
- name: C
  role: Implementation language of AST; native behaviour underlies the binding-level
    observations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:8165a795e661fbdd46bed1c4123fdef77df11116d308601b73a1d907f7c065d9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - C
  - Python
  - Starlink AST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# python-binding-api-exercising

## Summary

Exercises a C library's Python binding (specifically Starlink AST's FrameSet/Frame objects) to probe documented object semantics — such as Mapping behaviour and integrity restoration — through direct API mutation and observation.

## When to use

Use when a task requires verifying documented semantics of a library object model (e.g. FrameSet as Mapping vs as Frame) by driving its Python binding with scripted mutations and recording pass/fail behaviour.

## When NOT to use

- You need to build or install the library itself rather than exercise its API semantics — use the build/bootstrap workflow instead.
- The question concerns non-semantic properties (build configuration, compiler flags, F77/FC/CC environment) rather than object behaviour through the binding.
- The target library has no language bindings for the runtime you intend to drive (AST is written in C; its bindings cover Python, Java, Perl and Fortran).

## Inputs

- Starlink AST library installation (ascl:1404.016) with Python bindings available
- A FrameSet constructed with base Frame, intermediate Frame connected by a Mapping, and current Frame
- A target object attribute to mutate (e.g. a property of the current Frame)

## Outputs

- frameset_test_report.txt — parseable text recording pass/fail status for both the restoration-path check and the bypass-path check
- Observations of Mapping behaviour under each access path (restored vs unchanged Mappings)

## How to apply

First obtain the library — for AST this is the Starlink AST library (ascl:1404.016), written in C but with bindings for Python among other languages — and build it via the bootstrap sequence (./bootstrap, make configure-deps, ./configure -C, make world). Then instantiate the objects under test from Python: build a FrameSet containing a base Frame, an intermediate Frame connected by a Mapping, and a current Frame, and verify the FrameSet itself exposes the Mapping interface transforming coordinates from base to current Frame. Exercise the semantic under test through two access paths: mutate an attribute of the current Frame via the FrameSet reference (which should trigger automatic integrity restoration of the enclosed Mappings, Section 3.3), then repeat the identical mutation via a direct reference to the internal Frame object (which bypasses restoration). Record both paths' observed behaviour with explicit pass/fail status per check in a written report (e.g. frameset_test_report.txt).

## Related tools

- **Starlink AST** (Object model under test — FrameSet and Frame objects whose Mapping semantics and integrity restoration are exercised) — Starlink/starlink
- **Python** (Language binding used to instantiate FrameSet/Frame objects and script the mutation checks)
- **C** (Implementation language of AST; native behaviour underlies the binding-level observations)

## Examples

```
./bootstrap && make configure-deps && ./configure -C && make world
```

## Evaluation signals

- frameset_test_report.txt exists and is parseable text
- The report explicitly records the outcome of the restoration-path check (attribute mutation via FrameSet reference restores/simplifies the enclosed Mappings)
- The report explicitly records the outcome of the bypass-path check (direct internal Frame reference leaves Mappings unchanged)
- The FrameSet is confirmed to expose the Mapping interface transforming coordinates from the base Frame to the current Frame
- After mutation via the FrameSet reference, the Mapping still generates values matching the new current Frame

## Limitations

- Integrity restoration is automatic only through the FrameSet reference; obtaining a direct reference to an internal Frame bypasses it, leaving Mappings unchanged — this asymmetry must be tested on both paths.
- Bootstrap is a long-running step and patched autoconf/automake/libtool versions must be used; the OS-shipped versions will not work for building the tree.

## Evidence
