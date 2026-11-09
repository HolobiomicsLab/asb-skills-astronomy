---
name: region-based-mapping-routing
description: Use when a coordinate transformation must vary spatially across the coordinate
  space, and positions falling in different Regions need to be dispatched to different
  route Mappings via the Starlink AST library.
when_to_use_negative:
- The transformation is uniform across the whole coordinate space — a single Mapping
  inside the SwitchMap adds unnecessary complexity
- Regions overlap, since the selector Mapping requires disjoint Regions to produce
  an unambiguous route selection
- The target environment cannot build the Starlink AST library (e.g. no write access
  to STARCONF_DEFAULT_PREFIX or no compilers, since the build toolchain is required)
tools:
- name: Starlink AST
  role: Data model and library providing SwitchMap, SelectorMap, and Region classes
    for region-based coordinate dispatch
  repo: https://github.com/Starlink/starlink
- name: Python
  role: Binding language used to push test positions through the SwitchMap against
    the underlying C library
- name: C
  role: Language in which the AST library is written; the Python bindings wrap the
    C implementation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:7060b7184828c750220fdd26bd3b825dbe703907b267fa8e87e1f47a503323db
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
yes_rate: 0.75
n_claims: 8
---

# region-based-mapping-routing

## Summary

Reconstructs AST's SwitchMap pattern, in which a SelectorMap built from disjoint Regions uses its scalar selector output to dispatch each input position to one of several route Mappings. This lets a single compound Mapping apply different coordinate transforms in different areas of the coordinate space.

## When to use

Use when a coordinate transformation must vary spatially across the coordinate space, and positions falling in different Regions need to be dispatched to different route Mappings via the Starlink AST library.

## When NOT to use

- The transformation is uniform across the whole coordinate space — a single Mapping inside the SwitchMap adds unnecessary complexity
- Regions overlap, since the selector Mapping requires disjoint Regions to produce an unambiguous route selection
- The target environment cannot build the Starlink AST library (e.g. no write access to STARCONF_DEFAULT_PREFIX or no compilers, since the build toolchain is required)

## Inputs

- Starlink source checkout (Starlink/starlink) with AST component
- Set of disjoint Region definitions in the coordinate space
- One route Mapping per Region
- Table of test positions (including positions lying in no Region)

## Outputs

- Per-position routing table switchmap_routes.csv (position, containing Region, selected route Mapping or AST bad value)
- Selector index recorded for each test position

## How to apply

First build and install the AST component from the Starlink source checkout by running the manifest build (make /local-star/manifests/ast), which builds the AST library and everything it depends on under STARCONF_DEFAULT_PREFIX. Define a set of disjoint Regions in the coordinate space, then construct a SelectorMap from these Regions and combine it with one route Mapping per Region inside a SwitchMap. Push a table of test positions through the SwitchMap via the Python bindings to the underlying C library, recording which selector index (route Mapping) each position selects. Include positions lying in no Region: AST's data model specifies these are returned as the AST bad/null value rather than being routed. Validation: every in-Region test position must map through the route Mapping of its containing Region and all out-of-Region positions must yield the bad value; write per-position results (position, containing Region, selected route or bad value) to switchmap_routes.csv.

## Related tools

- **Starlink AST** (Data model and library providing SwitchMap, SelectorMap, and Region classes for region-based coordinate dispatch) — https://github.com/Starlink/starlink
- **Python** (Binding language used to push test positions through the SwitchMap against the underlying C library)
- **C** (Language in which the AST library is written; the Python bindings wrap the C implementation)

## Examples

```
make /local-star/manifests/ast
```

## Evaluation signals

- Every in-Region test position maps through the route Mapping of its containing Region
- All out-of-Region positions are returned as the AST bad/null value
- Selector index recorded per position matches the Region containing that position
- switchmap_routes.csv contains one row per test position with (position, containing Region, selected route or bad value)
- The AST component builds and installs successfully under STARCONF_DEFAULT_PREFIX via the manifest build

## Limitations

- Building a single Starlink component requires building everything it depends on, so the manifest build can be lengthy
- The full Starlink classic build sequence (./bootstrap, make configure-deps, ./configure -C, make world) is needed if the tree has not been bootstrapped; the patched autoconf/automake/libtool from buildsupport must precede system versions on PATH
- Stale Starlink environment variables (INSTALL, STARLINK) or an old-style Starlink system on PATH can confuse the build (e.g. messgen)
- Python is only one of several bindings (Java, Perl, Fortran also exist); the selector index table must be produced through an available binding to the C library

## Evidence

- [results] selector dispatch to route Mappings: [evidence span withheld — anchor status: not_found]
