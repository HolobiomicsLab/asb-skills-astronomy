---
name: ast-channel-text-serialisation
description: Use when you need to generate or validate the human-readable, block-structured
  AST text description of a Mapping (e.g. a CmpMap of PermMap and ZoomMap) and compare
  it against a reference listing.
when_to_use_negative:
- You need machine-readable serialisation (e.g. FITS-WCS encoding) rather than the
  Channel's human-readable block text — AST supports other storage formats for that
  purpose.
- The mappings of interest are not AST objects (e.g. arbitrary coordinate transforms
  outside the AST data model) — Channel only serialises AST Mappings such as PermMap,
  ZoomMap and CmpMap.
tools:
- name: Starlink AST
  role: Provides the Channel class that emits the block-structured text serialisation
    and the PermMap/ZoomMap/CmpMap mapping classes being serialised
  repo: https://github.com/Starlink/starlink
- name: Python
  role: Language in which the Section 4 example is run through AST bindings to instantiate
    the mappings and invoke the Channel
- name: Starlink AST library
  role: The underlying C library (ascl:1404.016) whose CmpMap/PermMap/ZoomMap data
    model produces the serialisation
  repo: https://github.com/Starlink/starlink
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:791fcb8ca84896cc1ff40543882ae2c70c2b86fb1978ded2a47829614009737b
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
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ast-channel-text-serialisation

## Summary

Reproduce the block-structured text serialisation that the Starlink AST library's Channel class emits for compound mappings, e.g. a CmpMap enclosing a PermMap and a ZoomMap, and verify it character-exactly against the listing printed in Section 4 of the AST paper (Berry, Warren-Smith & Jenness 2016).

## When to use

Use when you need to generate or validate the human-readable, block-structured AST text description of a Mapping (e.g. a CmpMap of PermMap and ZoomMap) and compare it against a reference listing.

## When NOT to use

- You need machine-readable serialisation (e.g. FITS-WCS encoding) rather than the Channel's human-readable block text — AST supports other storage formats for that purpose.
- The mappings of interest are not AST objects (e.g. arbitrary coordinate transforms outside the AST data model) — Channel only serialises AST Mappings such as PermMap, ZoomMap and CmpMap.

## Inputs

- Python Section 4 example code instantiating PermMap([3,1],[2,-1,1],12.2) and ZoomMap(3,4.0)
- Starlink AST library with Python bindings (starlink-pyast or source-built 'ast' component)
- Reference Channel listing printed in Section 4 of the AST paper

## Outputs

- cmpmap_channel_output.txt — the block-structured Channel serialisation of the CmpMap, saved verbatim
- output_matches_reference validation result (character-exact modulo insignificant whitespace)

## How to apply

Obtain the AST library with Python bindings — either pip-install starlink-pyast or build the 'ast' component from the Starlink source tree, e.g. via '% make /local-star/manifests/ast' after running ./bootstrap, make configure-deps, ./configure -C and make world. Instantiate the component mappings exactly as in the paper's Section 4 example: PermMap([3,1],[2,-1,1],12.2) and ZoomMap(3,4.0). Combine them with the CmpMap constructor, then feed the resulting CmpMap into an AST Channel to produce the block-structured listing (CmpMap with Nin=2/Nout=3 enclosing PermMap attributes Out1=2, Out2=-1, Out3=1, Con1=12.2 and ZoomMap with Zoom=4). Save the Channel text verbatim to cmpmap_channel_output.txt and diff it against the Section 4 listing; the check passes on a character-exact match modulo insignificant whitespace.

## Related tools

- **Starlink AST** (Provides the Channel class that emits the block-structured text serialisation and the PermMap/ZoomMap/CmpMap mapping classes being serialised) — https://github.com/Starlink/starlink
- **Python** (Language in which the Section 4 example is run through AST bindings to instantiate the mappings and invoke the Channel)
- **Starlink AST library** (The underlying C library (ascl:1404.016) whose CmpMap/PermMap/ZoomMap data model produces the serialisation) — https://github.com/Starlink/starlink

## Examples

```
import starlink.Ast as Ast; cm = Ast.CmpMap(Ast.PermMap([3,1],[2,-1,1],12.2), Ast.ZoomMap(3,4.0)); ch = Ast.Channel(cm); open('cmpmap_channel_output.txt','w').write(ch.read())
```

## Evaluation signals

- output_matches_reference: diff of cmpmap_channel_output.txt against the Section 4 listing matches character-exactly, modulo insignificant whitespace
- file_exists: cmpmap_channel_output.txt was produced on disk
- file_format_is: the saved output is the block-structured Channel text (CmpMap entry with Nin=2, Nout=3, enclosing PermMap Out1=2, Out2=-1, Out3=1, Con1=12.2 and ZoomMap Zoom=4)
- The serialised CmpMap reports Nin=2 and Nout=3, consistent with composing a 2-in/2-out PermMap with a 3-in/3-out ZoomMap as in the paper's example

## Limitations

- Rebuilding the Starlink 'ast' component from source requires the full bootstrap sequence (./bootstrap, make configure-deps, ./configure -C, make world); the bootstrap script must be run (not autoreconf manually) because autoconf/automake/libtool are patched specifically for this tree.
- Building the Starlink classic applications requires setting STARCONF_DEFAULT_STARLINK and STARCONF_DEFAULT_PREFIX (defaulting to /star) and removing stale INSTALL/STARLINK environment variables; old compiler settings (e.g. IRAF's F77 script) can cause opaque libtool build failures.
- The comparison tolerance is 'insignificant whitespace' only — substantive formatting differences from the reference listing will fail the output_matches_reference check.

## Evidence

- [intro] The Starlink AST library (Warren-Smith and Berry, 2013, ascl:1404.016) provides a generalised scheme for modelling, manipulating and storing inter-related coordinate systems.: "The Starlink AST library (Warren-Smith and Berry, 2013, ascl:1404.016) provides a generalised scheme for modelling, manipulating and storing inter-related coordinate systems."
- [methods] Build single component via manifest file: "% make /local-star/manifests/ast

   This will build, and install under STARCONF_DEFAULT_PREFIX, this component and _everything it depends on"
- [readme] Starlink build sequence from README: "   % ./bootstrap
   % make configure-deps
   % ./configure -C
   % make world"
- [intro] AST language bindings: "Whilst written in C, it has bindings for several other languages including Python, Java, Perl and Fortran."
