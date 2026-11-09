---
name: stdofrest-reference-frame-handling
description: Use when a spectral axis must be transformed between systems defined
  by different StdOfRest, System or Unit attributes - e.g. heliocentric radio velocity
  (VRAD) to LSR frequency (FREQ) - using AST SpecFrames.
when_to_use_negative:
- Both spectral frames already share identical System, Unit and StdOfRest attributes
  - no inter-system Mapping is required.
- AST cannot generate the Mapping (convert only produces one 'if possible'), e.g.
  the velocity/frequency frame lacks a RestFreq, so the conversion cannot be realised.
- The goal is general world-coordinate attachment or FITS-WCS retrieval for images/cubes
  rather than 1-D spectral reference-frame conversion.
edam_operation: http://edamontology.org/operation_3434
tools:
- name: Starlink AST
  role: Supplies the SpecFrame class and the convert method that generate the FrameSet-encapsulated
    Mapping between spectral systems differing in System, Unit and StdOfRest
  repo: https://github.com/Starlink/starlink
- name: Starlink AST library
  role: Underlying C library (ascl:1404.016) whose data model stores the inter-related
    coordinate systems serialised through an AST Channel to frameset_vrad_to_freq.ast
  repo: https://github.com/Starlink/starlink
- name: Python
  role: 'Driving language via the starlink.Ast bindings: creates the VRAD SpecFrame,
    deep-copies it into the FREQ/LSR frame, invokes convert and writes the Channel
    output'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:588c58eacf1ee012f754b6c138e334ce6894d617672a1b2441ee7bdd5c13c440
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - Python
  - Starlink AST
  - Starlink AST library
  edam_operation: http://edamontology.org/operation_3434
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# stdofrest-reference-frame-handling

## Summary

Generates a Mapping between two AST SpecFrames whose System, Unit and StdOfRest attributes differ - e.g. heliocentric radio velocity (System=VRAD, RestFreq=345.8 GHz) versus LSR frequency (System=FREQ, Unit=Hz). The convert method returns this Mapping inside a FrameSet that also contains copies of both Frames, and the result can be serialised through an AST Channel for reuse.

## When to use

Use when a spectral axis must be transformed between systems defined by different StdOfRest, System or Unit attributes - e.g. heliocentric radio velocity (VRAD) to LSR frequency (FREQ) - using AST SpecFrames.

## When NOT to use

- Both spectral frames already share identical System, Unit and StdOfRest attributes - no inter-system Mapping is required.
- AST cannot generate the Mapping (convert only produces one 'if possible'), e.g. the velocity/frequency frame lacks a RestFreq, so the conversion cannot be realised.
- The goal is general world-coordinate attachment or FITS-WCS retrieval for images/cubes rather than 1-D spectral reference-frame conversion.

## Inputs

- SpecFrame configured with System=VRAD and RestFreq=345.8 GHz (heliocentric radio-velocity spectral axis)
- Deep-copied SpecFrame reconfigured to System=FREQ, Unit=Hz, StdOfRest=LSR

## Outputs

- FrameSet whose base and current frames are linked by the velocity-to-frequency Mapping and include copies of both source SpecFrames
- Serialised FrameSet text file (frameset_vrad_to_freq.ast) written via an AST Channel

## How to apply

Import the Starlink AST library through its Python bindings (starlink.Ast) and create a SpecFrame configured with System=VRAD and RestFreq=345.8 GHz to define the heliocentric radio-velocity axis. Deep-copy that SpecFrame and modify the copy to System=FREQ, Unit=Hz and StdOfRest=LSR, so both target systems exist as first-class Frames rather than ad-hoc conversion formulas. Invoke convert on the VRAD frame with the FREQ frame as argument (frame1.convert(frame2)); AST generates, if possible, a Mapping between the two spectral coordinate systems and encapsulates it in a FrameSet that also includes copies of both Frames. Orient the transformation through the FrameSet's base and current frames (base = VRAD source, current = FREQ target) before applying it to spectral values. Serialise the FrameSet through an AST Channel and write the text output to frameset_vrad_to_freq.ast. Validate by re-reading the file with an AST Channel to recreate an equivalent FrameSet, confirming the base/current frames report System=VRAD and System=FREQ respectively.

## Related tools

- **Starlink AST** (Supplies the SpecFrame class and the convert method that generate the FrameSet-encapsulated Mapping between spectral systems differing in System, Unit and StdOfRest) — https://github.com/Starlink/starlink
- **Starlink AST library** (Underlying C library (ascl:1404.016) whose data model stores the inter-related coordinate systems serialised through an AST Channel to frameset_vrad_to_freq.ast) — https://github.com/Starlink/starlink
- **Python** (Driving language via the starlink.Ast bindings: creates the VRAD SpecFrame, deep-copies it into the FREQ/LSR frame, invokes convert and writes the Channel output)

## Examples

```
from starlink import Ast; import copy; f1 = Ast.SpecFrame('System=VRAD,RestFreq=345.8GHz'); f2 = copy.deepcopy(f1); f2.System='FREQ'; f2.Unit='Hz'; f2.StdOfRest='LSR'; fs = f1.convert(f2)
```

## Evaluation signals

- The serialisation file frameset_vrad_to_freq.ast exists after the AST Channel write.
- Re-reading frameset_vrad_to_freq.ast through an AST Channel recreates an equivalent FrameSet (round-trip fidelity).
- The FrameSet's base frame reports System=VRAD and its current frame reports System=FREQ.
- The returned FrameSet contains copies of both configured SpecFrames in addition to the velocity-to-frequency Mapping.
- Mapping generation fails (no usable FrameSet) when frames are incompletely specified - e.g. missing RestFreq - rather than yielding a silently wrong transform.

## Limitations

- convert generates a Mapping only 'if possible'; a SpecFrame with a missing RestFreq or otherwise incompatible System/StdOfRest configuration will not map.
- The workflow requires a deep copy of the source SpecFrame so the VRAD and FREQ/LSR configurations coexist as two distinct Frames; editing a single frame in place loses the source system.
- Building AST from the Starlink source is environment-sensitive: gfortran is supported only from GCC4.3 onwards, and stale STARLINK/INSTALL variables or an IRAF-set F77 script can break the ./bootstrap -> make configure-deps -> ./configure -C -> make world sequence.

## Evidence

- [other] AST's convert method on a SpecFrame generates, when possible, a Mapping between two spectral coordinate systems, returned inside a FrameSet holding copies of both Frames.: [evidence span withheld — anchor status: not_found]
