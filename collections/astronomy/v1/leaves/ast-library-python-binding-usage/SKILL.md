---
name: ast-library-python-binding-usage
description: Use when a task requires modelling, manipulating or storing inter-related
  coordinate systems with AST — especially building SwitchMap/SelectorMap region-based
  routing and pushing position tables through route Mappings.
when_to_use_negative:
- The task only needs FITS-WCS reading/writing without custom Mapping composition
  — use AST's dedicated WCS facilities directly instead of hand-building SwitchMaps
- The coordinate transform is a single fixed Mapping with no region-dependent branching
  — a plain Mapping suffices and a SwitchMap adds needless complexity
tools:
- name: Starlink AST library
  role: Provides the generalised scheme for modelling, manipulating and storing inter-related
    coordinate systems, including Region, SelectorMap and SwitchMap classes
- name: Python
  role: Binding layer used to push tables of test positions through the SwitchMap
    to the underlying C library
- name: C
  role: Language in which AST is written; the Python bindings call the C library
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:dc0e84b9f6cf74496dfb6ec642f53f4707c48b2c10ee7c0e219f159f691488d5
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# ast-library-python-binding-usage

## Summary

Builds and drives the Starlink AST coordinate-system library (ascl:1404.016) through its Python bindings to the underlying C library, e.g. to construct SwitchMap/SelectorMap route Mappings over disjoint Regions and dispatch test positions.

## When to use

Use when a task requires modelling, manipulating or storing inter-related coordinate systems with AST — especially building SwitchMap/SelectorMap region-based routing and pushing position tables through route Mappings.

## When NOT to use

- The task only needs FITS-WCS reading/writing without custom Mapping composition — use AST's dedicated WCS facilities directly instead of hand-building SwitchMaps
- The coordinate transform is a single fixed Mapping with no region-dependent branching — a plain Mapping suffices and a SwitchMap adds needless complexity

## Inputs

- Set of disjoint AST Regions defining the coordinate space
- One route Mapping per Region for the SwitchMap
- Table of test input positions (including positions in no Region)

## Outputs

- Per-position results: position, containing Region, selected route Mapping index, or AST bad/null value (e.g. switchmap_routes.csv)

## How to apply

First build and install the AST component from a Starlink source checkout with its manifest file (make /local-star/manifests/ast), which builds AST and everything it depends on under STARCONF_DEFAULT_PREFIX; a full-tree alternative is ./bootstrap, make configure-deps, ./configure -C, make world, plus ./build-modules.sh for Perl modules. Then, using the Python bindings to the C library, define a set of disjoint Regions in the coordinate space, construct a SelectorMap from those Regions, and combine it with one route Mapping per Region inside a SwitchMap. Push a table of test positions through the SwitchMap, recording which selector index (route Mapping) each position selects. Include positions lying in no Region: per the AST data model, the selector Mapping's scalar output selects a route Mapping, and the selected route Mapping transforms the input position to produce the Switch Map's output position, while out-of-Region positions return the AST bad/null value. Write per-position results (position, containing Region, selected route or bad value) to a CSV such as switchmap_routes.csv.

## Related tools

- **Starlink AST library** (Provides the generalised scheme for modelling, manipulating and storing inter-related coordinate systems, including Region, SelectorMap and SwitchMap classes)
- **Python** (Binding layer used to push tables of test positions through the SwitchMap to the underlying C library)
- **C** (Language in which AST is written; the Python bindings call the C library)

## Examples

```
make /local-star/manifests/ast
```

## Evaluation signals

- Every in-Region test position maps through the route Mapping of its containing Region (selector index matches the Region that contains the position)
- All out-of-Region positions yield the AST bad/null value, not a transformed coordinate
- switchmap_routes.csv contains one row per test position with position, containing Region, and selected route or bad value
- AST component installs under STARCONF_DEFAULT_PREFIX with all dependencies resolved via the manifest build

## Limitations

- Regions used for the SelectorMap must be disjoint; ambiguous coverage would break the selector-index interpretation
- Building from the Starlink tree requires the patched autoconf/automake/libtool installed by ./bootstrap; OS versions 'will _not_ work'
- Compilers forced via F77/FC/CC (e.g. an IRAF F77 wrapper) can break the build with libtool errors about 'unable to infer tagged configuration'

## Evidence

- [intro] The Starlink AST library (Warren-Smith and Berry, 2013, ascl:1404.016) provides a generalised scheme for modelling, manipulating and storing inter-related coordinate systems.: "The Starlink AST library (Warren-Smith and Berry, 2013, ascl:1404.016) provides a generalised scheme for modelling, manipulating and storing inter-related coordinate systems."
- [intro] Python bindings to the C library: "Whilst written in C, it has bindings for several other languages including Python, Java, Perl and Fortran."
- [methods] Single-component manifest build: "This will build, and install under STARCONF_DEFAULT_PREFIX, this component and _everything it depends on"
- [readme] Full-tree build sequence: "% ./bootstrap
% make configure-deps
% ./configure -C
% make world"
