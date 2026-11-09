---
name: ast-mapping-construction
description: Use when you need to build or verify an AST compound Mapping (e.g. a
  CmpMap over PermMap/ZoomMap components) or check its block-structured Channel text
  output against a reference listing such as the paper's Section 4 output.
when_to_use_negative:
- The coordinate information already exists in data files as FITS-WCS — AST's stated
  strength is retrieving and interpreting such formats, so parse the existing description
  rather than re-building Mappings by hand.
- The transformation needed is not expressible as axis permutation with constant substitution
  plus a uniform zoom — PermMap and ZoomMap cover only those cases, so other AST Mapping
  classes are required.
- The goal is building other Starlink components such as the Java applications (TOPCAT/STILTS/SPLAT)
  — those are built separately via ant per the starjava README and are unrelated to
  this Mapping-construction skill.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Starlink AST
  role: Core library supplying the Mapping classes (PermMap, ZoomMap, CmpMap) and
    the Channel serialiser that emits the block-structured text listing
  repo: https://github.com/Starlink/starlink
- name: starlink-pyast
  role: pip-installable distribution of AST's Python bindings used to instantiate
    the Mappings and Channel without a full Starlink source build
- name: Python
  role: Binding language in which the paper's Section 4 Mapping-construction example
    is written
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:063ad2a16b25ec630f3676de98705e838b7f1ddc81bd7babc6f80db1c6640573
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - Python
  - Starlink AST
  - starlink-pyast
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# ast-mapping-construction

## Summary

Constructs compound coordinate Mappings with the Starlink AST library's Python bindings — combining primitive Mappings such as PermMap([3,1],[2,-1,1],12.2) and ZoomMap(3,4.0) into a CmpMap — and serialises the result through an AST Channel into AST's block-structured text form for inspection, storage, or comparison against a reference listing.

## When to use

Use when you need to build or verify an AST compound Mapping (e.g. a CmpMap over PermMap/ZoomMap components) or check its block-structured Channel text output against a reference listing such as the paper's Section 4 output.

## When NOT to use

- The coordinate information already exists in data files as FITS-WCS — AST's stated strength is retrieving and interpreting such formats, so parse the existing description rather than re-building Mappings by hand.
- The transformation needed is not expressible as axis permutation with constant substitution plus a uniform zoom — PermMap and ZoomMap cover only those cases, so other AST Mapping classes are required.
- The goal is building other Starlink components such as the Java applications (TOPCAT/STILTS/SPLAT) — those are built separately via ant per the starjava README and are unrelated to this Mapping-construction skill.

## Inputs

- Python environment with the starlink-pyast bindings (or a locally built Starlink AST installation from the Starlink/starlink source tree)
- Mapping constructor parameters from the paper's Section 4 example: PermMap permutation lists [3,1] and [2,-1,1] with constants value 12.2; ZoomMap Nin=3 and Zoom=4.0
- Reference Channel listing printed in Section 4 of the AST paper (expected-output text used for diffing)

## Outputs

- AST CmpMap object with Nin=2, Nout=3 composing the PermMap and ZoomMap
- Block-structured AST Channel text serialisation saved to cmpmap_channel_output.txt, with a Begin/End CmpMap block enclosing Begin/End PermMap (Out1=2, Out2=-1, Out3=1, Con1=12.2) and Begin/End ZoomMap (Zoom=4) sub-blocks

## How to apply

Obtain AST via the pip-installable starlink-pyast Python bindings, or build only the 'ast' component from the Starlink source tree with '% make /local-star/manifests/ast', which installs that component and everything it depends on under STARCONF_DEFAULT_PREFIX (a full-tree build instead runs ./bootstrap; make configure-deps; ./configure -C; make world). Construct the primitive Mappings exactly as in the paper's Section 4 example: ast.PermMap([3,1],[2,-1,1],12.2), whose negative entry (-1) selects the constants vector to give Con1=12.2, and ast.ZoomMap(3,4.0), a Nin=3 Mapping with uniform scale factor 4.0. Combine the components with the AST CmpMap constructor into a single compound Mapping. Feed the CmpMap into an ast.Channel and save the emitted block-structured text verbatim to cmpmap_channel_output.txt. Validate by diffing the saved file against the exact Channel listing printed in Section 4 of the paper — the match should be character-exact modulo insignificant whitespace, showing a CmpMap (Nin=2, Nout=3) block enclosing a PermMap (Out1=2, Out2=-1, Out3=1, Con1=12.2) and a ZoomMap (Zoom=4). The rationale is that Mappings are the core of AST's data model for inter-related coordinate systems, and the Channel text form is AST's storable, human-readable serialisation of them.

## Related tools

- **Starlink AST** (Core library supplying the Mapping classes (PermMap, ZoomMap, CmpMap) and the Channel serialiser that emits the block-structured text listing) — https://github.com/Starlink/starlink
- **starlink-pyast** (pip-installable distribution of AST's Python bindings used to instantiate the Mappings and Channel without a full Starlink source build)
- **Python** (Binding language in which the paper's Section 4 Mapping-construction example is written)

## Examples

```
from starlink import ast; cmp = ast.CmpMap(ast.PermMap([3,1],[2,-1,1],12.2), ast.ZoomMap(3,4.0)); ast.Channel().write(cmp)
```

## Evaluation signals

- The Channel output opens with a 'Begin CmpMap' block carrying Nin = 2 and Nout = 3, and closes with the matching 'End CmpMap'.
- The nested PermMap block serialises Out1 = 2, Out2 = -1, Out3 = 1 and Con1 = 12.2, with the -1 entry resolving to the first element of the constants vector.
- The nested ZoomMap block serialises Zoom = 4, matching the 4.0 scale factor passed to ZoomMap(3, 4.0).
- diff of cmpmap_channel_output.txt against the Section 4 listing is character-exact modulo insignificant whitespace (output_matches_reference), and file_exists / file_format_is checks pass on the saved file.
- Re-running the construction with identical constructor arguments regenerates the identical listing after whitespace normalisation.

## Limitations

- Source builds of AST depend on Starlink's own patched autoconf, automake and libtool — versions shipped with the OS will not work — and the full ./bootstrap; make configure-deps; ./configure -C; make world sequence takes a long while; the pip-installable starlink-pyast route avoids this.
- The validation criterion is character-exact match only modulo insignificant whitespace, so indentation/spacing differences must be normalised before diffing the Channel output against the Section 4 listing.
- The skill exercises only PermMap (permutation with constant substitution, Con1 = 12.2) and ZoomMap (uniform scale 4.0) components; more complex coordinate transforms require other AST Mapping classes not covered here.

## Evidence

- [full_text] Channel listing structure: CmpMap encloses PermMap and ZoomMap: [evidence span withheld — anchor status: not_found]
- [full_text] Validation by diff against the paper's Section 4 Channel listing: [evidence span withheld — anchor status: not_found]
