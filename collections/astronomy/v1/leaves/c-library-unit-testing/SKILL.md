---
name: c-library-unit-testing
description: Use when a C library's documented per-class rules (e.g. Mapping simplification
  in AST Section 3.1.3) must be empirically validated by constructing representative
  inputs and asserting the class or value of each result.
when_to_use_negative:
- The library under test is already installed via a package manager and the goal is
  only to call it, not to validate rules — use it directly instead of rebuilding from
  the source checkout
- The behaviour of interest is not governed by per-class documented rules, so there
  is no expected result class to assert against
- The transformation pipeline involves Java-side (starjava/JNIAST) behaviour rather
  than the C AST library — build starjava with ant instead
tools:
- name: Starlink AST
  role: C library under test; provides astSimplify and the Mapping classes (CmpMap,
    UnitMap, MatrixMap, ShiftMap) whose simplification rules are verified
- name: C
  role: Language used to write the test program that constructs compound Mappings
    and checks simplified results against the AST library
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:c7d3d2d6b8c0e0dcf7dc48770b44213068480a0f113c534bca68bad708367c1d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C
  - Starlink AST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# C library unit testing

## Summary

Write and run C test programs against a compiled scientific C library (here, the Starlink AST library built from source) to verify that each documented behavioural rule—e.g. AST's CmpMap simplification rules—yields the expected result, recording pass/fail outcomes per rule.

## When to use

Use when a C library's documented per-class rules (e.g. Mapping simplification in AST Section 3.1.3) must be empirically validated by constructing representative inputs and asserting the class or value of each result.

## When NOT to use

- The library under test is already installed via a package manager and the goal is only to call it, not to validate rules — use it directly instead of rebuilding from the source checkout
- The behaviour of interest is not governed by per-class documented rules, so there is no expected result class to assert against
- The transformation pipeline involves Java-side (starjava/JNIAST) behaviour rather than the C AST library — build starjava with ant instead

## Inputs

- Starlink source checkout containing the AST component
- Manifest target /local-star/manifests/ast defining the component and its dependencies
- Documented rule set (AST paper Section 3.1.3 simplification rules) specifying the expected result class per case
- C test program source constructing compound Mappings (Mapping+inverse, compound with UnitMap, series/parallel MatrixMaps, series ShiftMaps)

## Outputs

- Per-rule pass/fail outcomes with the resulting Mapping class for each of the four astSimplify cases
- mapping_simplification_report: serialized per-rule results including expected vs. observed Mapping class
- Failure entries for any rule whose simplified Mapping class does not match the documented expectation

## How to apply

First build and install only the component under test from the Starlink source checkout using its manifest target, which installs the library under STARCONF_DEFAULT_PREFIX and puts the bin directory on the PATH. Then write a C test program linked against the installed AST library that constructs one concrete object per documented rule — here the four compound-Mapping cases: a Mapping compounded with its own inverse, a compound containing an embedded UnitMap, two MatrixMaps in series and in parallel, and two adjacent ShiftMaps in series. Apply the API call that exercises the rule (astSimplify on each compound CmpMap) and inspect the class of each resulting Mapping with the library's introspection facilities. Record pass/fail per rule, where pass means the simplified Mapping class matches the class stated in Section 3.1.3 (Mapping+inverse collapses to UnitMap; UnitMaps removed; series/parallel MatrixMaps merge into a single MatrixMap; adjacent ShiftMaps merge into one ShiftMap). Finally serialize the per-rule outcomes, including the resulting Mapping class per case, into a machine-readable report so any mismatch is captured as a failure entry.

## Related tools

- **Starlink AST** (C library under test; provides astSimplify and the Mapping classes (CmpMap, UnitMap, MatrixMap, ShiftMap) whose simplification rules are verified)
- **C** (Language used to write the test program that constructs compound Mappings and checks simplified results against the AST library)

## Examples

```
make /local-star/manifests/ast && gcc test_simplify.c -o test_simplify -last -lgrf3d && ./test_simplify
```

## Evaluation signals

- Mapping compounded with its own inverse simplifies (via astSimplify) to a UnitMap
- UnitMaps embedded in a compound are removed by astSimplify
- Two MatrixMaps compounded in series, and two in parallel, each merge into a single MatrixMap
- Two adjacent ShiftMaps in series merge into one ShiftMap
- Every rule mismatch is recorded as a failure entry in mapping_simplification_report rather than silently passing

## Limitations

- Simplification is class-specific: each class of Mapping provides its own rules, so new or unmapped classes may pass through astSimplify unchanged rather than collapsing
- Building only the AST component still builds and installs everything it depends on, so environment variables (STARCONF_DEFAULT_PREFIX, PATH) and stale installs can confuse the build
- Validation is restricted to the four expected classes stated in Section 3.1.3; any other observed class counts as a failure even if functionally correct

## Evidence
