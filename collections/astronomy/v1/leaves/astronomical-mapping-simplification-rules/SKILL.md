---
name: astronomical-mapping-simplification-rules
description: Use when a compound AST Mapping (CmpMap) built by stacking transformations
  must be reduced to its canonical minimal form, and each class-specific simplification
  rule must be empirically verified.
when_to_use_negative:
- The Mapping under test is a single (non-compound) Mapping with no adjacent Mapping
  to combine with, so no simplification rule applies
- The goal is coordinate transformation itself (applying a Mapping to data) rather
  than inspecting or reducing the Mapping's structure
- Working outside the AST library's data model, where 'simplification' semantics of
  other coordinate-system frameworks are not governed by AST's per-class rules
tools:
- name: Starlink AST
  role: Library whose CmpMap, UnitMap, MatrixMap, ShiftMap classes and astSimplify
    function supply the simplification rules under test
  repo: https://github.com/Starlink/starlink
- name: C
  role: Language in which the AST test program is written, since AST is written in
    C
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:72c070eaa24d3517cb2579ee4998b989869514b6fb694939b68dd825ccd444dc
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
yes_rate: 1.0
n_claims: 13
---

# astronomical-mapping-simplification-rules

## Summary

Reconstructs and verifies the CmpMap simplification and merge rules of the Starlink AST Mapping class by building AST from source and exercising astSimplify on controlled compound Mappings in C.

## When to use

Use when a compound AST Mapping (CmpMap) built by stacking transformations must be reduced to its canonical minimal form, and each class-specific simplification rule must be empirically verified.

## When NOT to use

- The Mapping under test is a single (non-compound) Mapping with no adjacent Mapping to combine with, so no simplification rule applies
- The goal is coordinate transformation itself (applying a Mapping to data) rather than inspecting or reducing the Mapping's structure
- Working outside the AST library's data model, where 'simplification' semantics of other coordinate-system frameworks are not governed by AST's per-class rules

## Inputs

- Starlink source checkout containing the AST component
- C test program constructing compound Mappings: Mapping+inverse CmpMap, CmpMap with embedded UnitMap, series and parallel MatrixMap pairs, and series ShiftMap pair
- Section 3.1.3 of the AST paper defining the expected simplified Mapping classes

## Outputs

- mapping_simplification_report with per-rule pass/fail outcomes and the resulting Mapping class for each of the four cases
- Simplified Mapping objects returned by astSimplify for each compound case

## How to apply

Build and install the AST component alone from the Starlink source checkout using its manifest target (make /local-star/manifests/ast), which installs under STARCONF_DEFAULT_PREFIX and adds the bin directory to the PATH. Write a C test program against the AST library that constructs the four compound cases from Section 3.1.3: a Mapping compounded with its own inverse, a compound containing an embedded UnitMap, two MatrixMaps compounded in series and in parallel, and two adjacent ShiftMaps in series. Apply astSimplify to each compound Mapping and inspect the class of each simplified result — Mapping+inverse should collapse to UnitMap, embedded UnitMaps should be removed from compounds, series/parallel MatrixMaps should merge into a single MatrixMap, and adjacent ShiftMaps in series should merge into one ShiftMap. Record pass/fail per rule and serialize the resulting Mapping class for each case into mapping_simplification_report. The rationale is that each Mapping class supplies its own rules indicating when and how it can be simplified or combined with an adjacent Mapping in the chain, so the rule set must be probed per class.

## Related tools

- **Starlink AST** (Library whose CmpMap, UnitMap, MatrixMap, ShiftMap classes and astSimplify function supply the simplification rules under test) — https://github.com/Starlink/starlink
- **C** (Language in which the AST test program is written, since AST is written in C)

## Examples

```
Mapping *simpl = astSimplify( cmpMap ); const char *cls = astGetC( simpl, "Class" );
```

## Evaluation signals

- A Mapping compounded with its own inverse simplifies to a UnitMap
- UnitMaps embedded within a compound Mapping are removed after astSimplify
- Two MatrixMaps compounded in series and in parallel merge into a single MatrixMap
- Two adjacent ShiftMaps in series merge into one ShiftMap
- Each of the four rules must yield the expected Mapping class stated in Section 3.1.3; any mismatch is recorded as a failure entry in the report

## Limitations

- Each class of Mapping provides its own simplification rules, so rules learned for one class (e.g. ShiftMap) cannot be assumed to transfer to other Mapping classes
- Building AST from source requires the full bootstrap sequence (./bootstrap, make configure-deps, ./configure -C, make world) with patched autoconf/automake/libtool installed into $STARCONF_DEFAULT_PREFIX/buildsupport/bin before ./bootstrap runs
- Verification is limited to the four compound cases enumerated in Section 3.1.3; other compound configurations are not covered by this report

## Evidence
