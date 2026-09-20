---
name: reference-output-diff-comparison
description: Use when a paper prints a human-readable reference listing (e.g. an AST
  Channel block-structured serialisation) and the goal is to reproduce that exact
  text with library code and validate it by diff.
when_to_use_negative:
- The paper provides no reference listing or concrete parameter values to reconstruct
  — there is nothing to diff against.
- The expected output is binary or contains timestamps/version-dependent noise, making
  a character-exact diff meaningless.
- The task is to validate FITS-WCS round-tripping or coordinate transformation behaviour
  rather than to reproduce a printed serialisation listing.
tools:
- name: Starlink AST
  role: Library whose Channel class produces the block-structured text serialisation
    of the CmpMap/PermMap/ZoomMap objects to be diffed against the paper's listing
  repo: https://github.com/Starlink/starlink
- name: Python
  role: Binding language used to instantiate the mappings and run the Section 4 example
    through the AST Channel
- name: Starlink AST library
  role: Source distribution to bootstrap and build (or install as starlink-pyast)
    to obtain the AST Channel functionality
  repo: https://github.com/Starlink/starlink
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:c00f7e736169d4e290bd27388f2aeaf529603c07c29463c39830dfd3f7f186b1
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
yes_rate: 1.0
n_claims: 9
---

# reference-output-diff-comparison

## Summary

Reproduce a reference text listing printed in a paper (here, the AST Channel serialisation of a CmpMap of a PermMap and ZoomMap from Section 4 of Berry et al. 2016) by running the paper's example through the actual library, saving the output verbatim, and diffing it against the published listing.

## When to use

Use when a paper prints a human-readable reference listing (e.g. an AST Channel block-structured serialisation) and the goal is to reproduce that exact text with library code and validate it by diff.

## When NOT to use

- The paper provides no reference listing or concrete parameter values to reconstruct — there is nothing to diff against.
- The expected output is binary or contains timestamps/version-dependent noise, making a character-exact diff meaningless.
- The task is to validate FITS-WCS round-tripping or coordinate transformation behaviour rather than to reproduce a printed serialisation listing.

## Inputs

- Starlink AST library with Python bindings (starlink-pyast or a source build of the 'ast' component)
- Component mapping constructor parameters from the paper: PermMap([3,1],[2,-1,1],12.2) and ZoomMap(3,4.0)
- The reference Channel listing printed in Section 4 of the paper (AST: A library for modelling and manipulating coordinate systems, DOI 10.1016/j.ascom.2016.02.003)

## Outputs

- cmpmap_channel_output.txt — the block-structured Channel serialisation of the CmpMap, saved verbatim
- A diff verdict indicating character-exact match (modulo insignificant whitespace) against the Section 4 listing

## How to apply

Obtain the Starlink AST library with Python bindings (pip-installable starlink-pyast, or build the 'ast' component from the Starlink source tree — the full build sequence is ./bootstrap, make configure-deps, ./configure -C, make world, and a single component can be built via its manifest file, e.g. make /local-star/manifests/ast). Instantiate the paper's Section 4 component mappings, PermMap([3,1],[2,-1,1],12.2) and ZoomMap(3,4.0), combine them with the AST CmpMap constructor, and feed the resulting CmpMap into an AST Channel to generate the block-structured human-readable serialisation. Save the Channel text verbatim to cmpmap_channel_output.txt, then diff it character-exactly (modulo insignificant whitespace) against the listing printed in Section 4. The check is meaningful because Channel output is deterministic given the same object attributes (CmpMap Nin=2/Nout=3, PermMap Out1=2/Out2=-1/Out3=1/Con1=12.2, ZoomMap Zoom=4).

## Related tools

- **Starlink AST** (Library whose Channel class produces the block-structured text serialisation of the CmpMap/PermMap/ZoomMap objects to be diffed against the paper's listing) — https://github.com/Starlink/starlink
- **Python** (Binding language used to instantiate the mappings and run the Section 4 example through the AST Channel)
- **Starlink AST library** (Source distribution to bootstrap and build (or install as starlink-pyast) to obtain the AST Channel functionality) — https://github.com/Starlink/starlink

## Examples

```
from starlink import ast; cmpmap = ast.CmpMap(ast.PermMap([3,1],[2,-1,1],12.2), ast.ZoomMap(3,4.0)); print(ast.Channel().write(cmpmap))
```

## Evaluation signals

- output_matches_reference: character-exact diff of cmpmap_channel_output.txt against the Section 4 listing, modulo insignificant whitespace
- file_exists: cmpmap_channel_output.txt was created by the Channel run
- file_format_is: the saved file is plain block-structured Channel text
- Structural attributes appear in the output: CmpMap with Nin=2, Nout=3 enclosing a PermMap (Out1=2, Out2=-1, Out3=1, Con1=12.2) and a ZoomMap (Zoom=4)
- The PermMap/ZoomMap constructor arguments exactly match the paper's Section 4 example ([3,1]/[2,-1,1] with constant 12.2; zoom factor 3, value 4.0)

## Limitations

- Building AST from the Starlink source tree requires patched autoconf/automake/libtool installed by ./bootstrap — system-shipped versions will not work — and the bootstrap/build step takes a long while.
- The comparison tolerates only insignificant whitespace; any version-dependent changes in Channel output formatting could break an exact diff.
- gfortran is only supported from GCC 4.3 onwards for the classic Starlink source build; the Java components (starjava) are built separately and are not needed for this Python-based check.

## Evidence
