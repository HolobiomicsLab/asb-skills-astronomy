---
name: bad-value-handling-in-coordinate-transforms
description: Use when transforming test positions through an AST SwitchMap (or any
  Region-gated Mapping) and you must classify which route Mapping each input selects
  and which inputs yield the AST bad value.
when_to_use_negative:
- The Mapping is a simple unary transform (e.g. a WinMap or CmpMap) with no Region-based
  selector — bad-value routing semantics of SelectorMap do not apply
- Input positions are guaranteed to lie inside a single Region, so no out-of-domain
  positions need be detected
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Starlink AST
  role: Library providing the SwitchMap, SelectorMap, and Region classes used to dispatch
    input positions to route Mappings and return the bad value for out-of-Region inputs
- name: Starlink AST library
  role: Generalised scheme for modelling, manipulating and storing inter-related coordinate
    systems in which the SelectorMap/SwitchMap test is implemented
- name: Python
  role: Binding language used to push the table of test positions through the SwitchMap
    to the underlying C library
- name: C
  role: Language in which the AST library is written; the Python bindings call into
    it
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:04a643d26f1fd978afe3179aa0b16fc06139b78e8a393e816a4486e3c2e6adeb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C
  - Python
  - Starlink AST
  - Starlink AST library
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bad-value-handling-in-coordinate-transforms

## Summary

A method for handling out-of-domain positions in AST coordinate transforms: positions that fall outside every Region used by a SelectorMap/SwitchMap must be returned as the AST bad/null value rather than an arbitrary or clamped coordinate.

## When to use

Use when transforming test positions through an AST SwitchMap (or any Region-gated Mapping) and you must classify which route Mapping each input selects and which inputs yield the AST bad value.

## When NOT to use

- The Mapping is a simple unary transform (e.g. a WinMap or CmpMap) with no Region-based selector — bad-value routing semantics of SelectorMap do not apply
- Input positions are guaranteed to lie inside a single Region, so no out-of-domain positions need be detected

## Inputs

- AST SwitchMap composed of a SelectorMap built from disjoint Regions plus one route Mapping per Region
- Table of test input positions in the coordinate space, including positions lying in no Region
- Starlink source checkout with the ast component manifest

## Outputs

- switchmap_routes.csv with per-position records: input position, containing Region, selected route Mapping index, or AST bad/null value
- Classification of every test position as in-Region (routed) or out-of-Region (bad value)

## How to apply

Build and install the AST component from the Starlink source checkout via its manifest file (`make /local-star/manifests/ast`), which builds the component and everything it depends on. Define a set of disjoint Regions in the coordinate space using the AST library, construct a SelectorMap from those Regions, and combine it with one route Mapping per Region inside a SwitchMap. Push a table of test positions through the SwitchMap via the Python bindings to the underlying C library, recording the selector index (route Mapping) chosen for each position. Deliberately include positions lying in no Region and confirm they come back as the AST bad/null value: in the AST data model, the selector Mapping's scalar output selects a route Mapping, and the selected route Mapping then transforms the input position to produce the Switch Map's output position, while unselected/out-of-Region inputs map to the bad value. Write per-position results (position, containing Region, selected route or bad value) to switchmap_routes.csv and validate against the two invariants.

## Related tools

- **Starlink AST** (Library providing the SwitchMap, SelectorMap, and Region classes used to dispatch input positions to route Mappings and return the bad value for out-of-Region inputs)
- **Starlink AST library** (Generalised scheme for modelling, manipulating and storing inter-related coordinate systems in which the SelectorMap/SwitchMap test is implemented)
- **Python** (Binding language used to push the table of test positions through the SwitchMap to the underlying C library)
- **C** (Language in which the AST library is written; the Python bindings call into it)

## Examples

```
% make /local-star/manifests/ast
```

## Evaluation signals

- Every in-Region test position maps through the route Mapping of its containing Region
- All out-of-Region test positions yield the AST bad/null value, not an extrapolated or clamped coordinate
- The selector index recorded for each position equals the index of the Region that contains that position
- switchmap_routes.csv contains one row per input position with a containing-Region or bad-value field for each
- The AST component was built successfully via `make /local-star/manifests/ast`, which installs the component and everything it depends on

## Limitations

- Regions used to build the SelectorMap must be disjoint; overlapping Regions make selector assignment ambiguous
- The bootstrap/build sequence downloads third-party software and patched autoconf/automake/libtool, so OS-shipped versions of those tools will not work, and a default IRAF installation's F77 script can cause the build to fail
- Positions in no Region are reported only as the AST bad/null value, so downstream code must check for the bad value explicitly

## Evidence

- [other] In AST's Switch Map, the selector Mapping's scalar output selects a route Mapping, and the selected route Mapping then transforms the input position to produce the Switch Map's output position.: [evidence span withheld — anchor status: not_found]
- [other] Include positions lying in no Region and confirm they are returned as the AST bad/null value.: [evidence span withheld — anchor status: not_found]
