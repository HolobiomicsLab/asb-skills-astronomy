---
name: switchmap-selectormap-construction
description: Use when input positions must be transformed differently depending on
  which disjoint Region of a coordinate space they fall in, and AST Mappings must
  be composed into a piecewise transform.
when_to_use_negative:
- Positions need a single uniform transformation across the whole coordinate space
  — a plain compound Mapping (e.g. CmpMap) suffices, not region-based routing
- The Regions overlap, since the SelectorMap routing assumes disjoint Regions and
  selector indices would be ambiguous
- Working outside the AST data model, e.g. writing custom interpolation code instead
  of composing AST Mappings
tools:
- name: Starlink AST
  role: Library whose data model supplies Region, SelectorMap and SwitchMap classes
    for region-based coordinate routing
  repo: Starlink/starlink
- name: Starlink AST library
  role: Generalised scheme for modelling, manipulating and storing inter-related coordinate
    systems; provides the Mappings composed here
  repo: Starlink/starlink
- name: Python
  role: Binding language used to push test position tables through the SwitchMap to
    the underlying C library
- name: C
  role: Language in which the AST library is written; invoked through the Python bindings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:a0cf97f9ea0238ff343e2a96f4d5209e6f28607b3cbdfd63d0e9f977c9110385
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - C
  - Python
  - Starlink AST
  - Starlink AST library
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# switchmap-selectormap-construction

## Summary

Reconstructs region-based coordinate routing in the Starlink AST library by building a SwitchMap whose SelectorMap dispatches each input position to one of several route Mappings based on which Region contains it.

## When to use

Use when input positions must be transformed differently depending on which disjoint Region of a coordinate space they fall in, and AST Mappings must be composed into a piecewise transform.

## When NOT to use

- Positions need a single uniform transformation across the whole coordinate space — a plain compound Mapping (e.g. CmpMap) suffices, not region-based routing
- The Regions overlap, since the SelectorMap routing assumes disjoint Regions and selector indices would be ambiguous
- Working outside the AST data model, e.g. writing custom interpolation code instead of composing AST Mappings

## Inputs

- Set of disjoint Region objects defined in a coordinate space (AST library)
- One route Mapping per Region (AST Mapping objects)
- Table of test input positions to transform

## Outputs

- SwitchMap compound Mapping dispatching per-Region transforms
- Per-position results (position, containing Region, selected route or bad value) written to switchmap_routes.csv

## How to apply

Build and install the AST component from the Starlink source checkout via its manifest file (make /local-star/manifests/ast), which builds the component and everything it depends on, then use the Python bindings to the underlying C library. Define a set of disjoint Regions in the coordinate space, construct a SelectorMap from those Regions, and combine it with one route Mapping per Region inside a SwitchMap. Push a table of test positions through the SwitchMap, recording which selector index (route Mapping) each position selects. Include positions lying in no Region to confirm they are returned as the AST bad/null value rather than being routed. The selector Mapping's scalar output selects the route Mapping, and the selected route Mapping transforms the input position to produce the SwitchMap's output position.

## Related tools

- **Starlink AST** (Library whose data model supplies Region, SelectorMap and SwitchMap classes for region-based coordinate routing) — Starlink/starlink
- **Starlink AST library** (Generalised scheme for modelling, manipulating and storing inter-related coordinate systems; provides the Mappings composed here) — Starlink/starlink
- **Python** (Binding language used to push test position tables through the SwitchMap to the underlying C library)
- **C** (Language in which the AST library is written; invoked through the Python bindings)

## Examples

```
make /local-star/manifests/ast
```

## Evaluation signals

- Every in-Region test position maps through the route Mapping of its containing Region and produces the expected output position
- All out-of-Region positions yield the AST bad/null value rather than a routed coordinate
- Each recorded selector index corresponds to the Region that geometrically contains the input position
- switchmap_routes.csv contains one row per test position with position, containing Region, and selected route or bad value
- The AST component and all its dependencies install successfully under STARCONF_DEFAULT_PREFIX via the manifest build

## Limitations

- Positions falling in no Region are returned as the AST bad/null value, so downstream code must handle bad values explicitly
- The manifest build (make /local-star/manifests/ast) rebuilds the component and everything it depends on, which can be lengthy
- Routing depends on Regions being well-defined in the coordinate space; degenerate or overlapping Region definitions would make selector assignment ambiguous

## Evidence
