---
name: compound-mapping-construction
description: Use when stacked coordinate transformations must be reduced or merged
  — e.g. a Mapping compounded with its own inverse, embedded UnitMaps, series/parallel
  MatrixMaps, or adjacent ShiftMaps — and the resulting simplified Mapping class must
  be verified.
when_to_use_negative:
- Input is a single, already-simple Mapping with no compound structure — astSimplify
  has nothing to merge and the four rules cannot be exercised.
- The goal is coordinate-system I/O such as attaching or reading FITS-WCS from astronomical
  data rather than simplifying Mapping chains.
tools:
- name: Starlink AST
  role: Provides the CmpMap/Mapping classes and the astSimplify routine used to construct
    compound transformations and collapse them by per-class rules
  repo: Starlink/starlink
- name: C
  role: Language in which the AST test program constructing the four compound cases
    is written
  repo: Starlink/starlink
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:67c2a999bde0091bf63191567b706df5655aa4c8219b9ed2a60ceb02ec98912e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - C
  - Starlink AST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# compound-mapping-construction

## Summary

A verification skill for the Starlink AST library that constructs compound Mappings (CmpMap) of increasing complexity and checks that astSimplify collapses them according to the per-class simplification and merge rules described in the AST data-model paper.

## When to use

Use when stacked coordinate transformations must be reduced or merged — e.g. a Mapping compounded with its own inverse, embedded UnitMaps, series/parallel MatrixMaps, or adjacent ShiftMaps — and the resulting simplified Mapping class must be verified.

## When NOT to use

- Input is a single, already-simple Mapping with no compound structure — astSimplify has nothing to merge and the four rules cannot be exercised.
- The goal is coordinate-system I/O such as attaching or reading FITS-WCS from astronomical data rather than simplifying Mapping chains.

## Inputs

- Compound Mapping (CmpMap) configurations built with the AST C API: Mapping compounded with its own inverse
- CmpMap containing an embedded UnitMap
- Two MatrixMaps compounded in series and in parallel
- Two adjacent ShiftMaps in series
- AST library built and installed under STARCONF_DEFAULT_PREFIX

## Outputs

- mapping_simplification_report with per-rule pass/fail outcomes
- Resulting (simplified) Mapping class for each of the four compound cases (expected: UnitMap, compound minus UnitMaps, single MatrixMap, single ShiftMap)

## How to apply

Build and install the AST component alone from the Starlink source checkout using its manifest target (make /local-star/manifests/ast), which installs under STARCONF_DEFAULT_PREFIX and adds the bin directory to the PATH. Write a C test program against the AST library that constructs the four compound cases: a Mapping compounded with its own inverse, a compound containing an embedded UnitMap, two MatrixMaps compounded in series and in parallel, and two adjacent ShiftMaps in series. Apply astSimplify to each compound Mapping, then inspect the class of each simplified Mapping. Each class of Mapping provides its own rules indicating when and how it can be simplified or combined with an adjacent Mapping in the chain, so the expected outcomes are: Mapping+inverse collapses to UnitMap, UnitMaps are removed from compounds, series/parallel MatrixMaps merge into a single MatrixMap, and adjacent ShiftMaps merge into one ShiftMap. Record pass/fail per rule, including the resulting Mapping class for each case, into a mapping_simplification_report.

## Related tools

- **Starlink AST** (Provides the CmpMap/Mapping classes and the astSimplify routine used to construct compound transformations and collapse them by per-class rules) — Starlink/starlink
- **C** (Language in which the AST test program constructing the four compound cases is written) — Starlink/starlink

## Examples

```
make /local-star/manifests/ast
```

## Evaluation signals

- Mapping compounded with its own inverse simplifies to a UnitMap
- UnitMaps are removed from a compound Mapping after astSimplify
- Two MatrixMaps compounded in series (and in parallel) merge into a single MatrixMap
- Two adjacent ShiftMaps in series merge into one ShiftMap
- Any simplified Mapping whose class differs from the expected class in Section 3.1.3 is recorded as a failure entry in the report

## Limitations

- Simplification behaviour is defined per Mapping class; a rule only holds when the expected class is produced, and any mismatch must be recorded as a failure rather than assumed acceptable.
- Requires building AST from the Starlink source checkout via its manifest target; the bootstrap installs patched autoconf/automake/libtool into buildsupport, so STARCONF_DEFAULT_PREFIX/bin and buildsupport/bin must be on the PATH before running ./bootstrap.
- Verification is class-based inspection only; it checks the type of the simplified Mapping, not numerical coordinate values.

## Evidence

- [other] Apply astSimplify to each compound Mapping.: [evidence span withheld — anchor status: not_found]
