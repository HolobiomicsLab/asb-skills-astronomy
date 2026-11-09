---
name: python-bindings-execution
description: Use when the goal is to reproduce AST paper examples (e.g. the Section
  4 CmpMap of PermMap([3,1],[2,-1,1],12.2) and ZoomMap(3,4.0)) by running them programmatically
  via Python rather than building C code by hand.
when_to_use_negative:
- The task only needs AST concepts explained conceptually — no executable reproduction
  of a mapping serialisation is required.
- The target environment cannot install the Starlink AST library or its Python bindings,
  and building the 'ast' component via the Starlink manifest is not an option.
- The serialisation target is a different AST object than the CmpMap-of-PermMap-and-ZoomMap
  listing covered by the paper's Section 4 reference output.
tools:
- name: Starlink AST
  role: Library whose AST objects (CmpMap, PermMap, ZoomMap, Channel) are constructed
    and serialised through the Python bindings
  repo: Starlink/starlink
- name: Python
  role: Binding language used to execute the paper's Section 4 example against the
    AST library
  repo: Starlink/starlink
- name: Starlink AST library
  role: Underlying C library (ascl:1404.016) providing the generalised scheme for
    modelling and storing coordinate systems
  repo: Starlink/starlink
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:75518b747c18b71a72e0b2912fdd0e58816bfd039d8bf3e8a4a4c40b5f64f156
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - Python
  - Starlink AST
  - Starlink AST library
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# python-bindings-execution

## Summary

Execute the Starlink AST library through its Python bindings (starlink-pyast) to instantiate mappings such as PermMap and ZoomMap, combine them into a CmpMap, and serialise the result with an AST Channel. The article notes AST is written in C but ships bindings for Python, Java, Perl and Fortran.

## When to use

Use when the goal is to reproduce AST paper examples (e.g. the Section 4 CmpMap of PermMap([3,1],[2,-1,1],12.2) and ZoomMap(3,4.0)) by running them programmatically via Python rather than building C code by hand.

## When NOT to use

- The task only needs AST concepts explained conceptually — no executable reproduction of a mapping serialisation is required.
- The target environment cannot install the Starlink AST library or its Python bindings, and building the 'ast' component via the Starlink manifest is not an option.
- The serialisation target is a different AST object than the CmpMap-of-PermMap-and-ZoomMap listing covered by the paper's Section 4 reference output.

## Inputs

- Python script instantiating AST mappings (PermMap with Out=[3,1], Con=[2,-1,1], Con1=12.2; ZoomMap(3,4.0))
- Starlink AST library with Python bindings (pip package starlink-pyast, or the 'ast' component built from the Starlink source tree)
- Reference Channel listing from Section 4 of the paper for comparison

## Outputs

- cmpmap_channel_output.txt — block-structured AST Channel text serialisation of the CmpMap (Nin=2, Nout=3) enclosing the PermMap and ZoomMap entries
- Boolean comparison result: output_matches_reference (character-exact modulo insignificant whitespace), plus file_exists and file_format_is checks

## How to apply

Install the pip-installable starlink-pyast bindings, or build the 'ast' component from the Starlink source tree via its manifest (make /local-star/manifests/ast after ./bootstrap, make configure-deps, ./configure -C, make world). Using the Python bindings, instantiate PermMap([3,1],[2,-1,1],12.2) and ZoomMap(3,4.0), then pass both to the AST CmpMap constructor to form the compound mapping. Feed the CmpMap into an AST Channel to generate the block-structured human-readable text serialisation, and save it verbatim (e.g. to cmpmap_channel_output.txt). Validate by diffing against the reference listing printed in Section 4 of the paper, expecting a character-exact match modulo insignificant whitespace, with the CmpMap entry (Nin=2, Nout=3) enclosing Out1=2, Out2=-1, Out3=1, Con1=12.2 for the PermMap and Zoom=4 for the ZoomMap.

## Related tools

- **Starlink AST** (Library whose AST objects (CmpMap, PermMap, ZoomMap, Channel) are constructed and serialised through the Python bindings) — Starlink/starlink
- **Python** (Binding language used to execute the paper's Section 4 example against the AST library) — Starlink/starlink
- **Starlink AST library** (Underlying C library (ascl:1404.016) providing the generalised scheme for modelling and storing coordinate systems) — Starlink/starlink

## Examples

```
from starlink import ast; cm = ast.CmpMap(ast.PermMap([3,1],[2,-1,1],12.2), ast.ZoomMap(3,4.0)); print(ast.Channel(cm))
```

## Evaluation signals

- cmpmap_channel_output.txt exists and is a text file (file_exists and file_format_is checks pass).
- Diff of the saved Channel output against the Section 4 listing yields a character-exact match, modulo insignificant whitespace (output_matches_reference).
- The serialised listing begins with a CmpMap entry with Nin=2, Nout=3 enclosing a PermMap (Out1=2, Out2=-1, Out3=1, Con1=12.2) and a ZoomMap (Zoom=4).
- The starlink-pyast import succeeds, or the 'ast' component builds cleanly via its manifest in the Starlink source tree.

## Limitations

- Building AST from source requires the full Starlink bootstrap sequence (./bootstrap, make configure-deps, ./configure -C, make world), which is slow; the pip-installable starlink-pyast is the lighter path.
- Channel output comparisons tolerate only insignificant whitespace — formatting differences beyond that will fail the character-exact check.
- The skill is anchored to the AST data model (mappings, Frames, Channels) and does not generalise to other coordinate-system libraries.

## Evidence
