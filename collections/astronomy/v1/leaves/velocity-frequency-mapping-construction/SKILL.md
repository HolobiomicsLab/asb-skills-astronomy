---
name: velocity-frequency-mapping-construction
description: Use when spectral data must be transformed between incompatible spectral
  coordinate systems (e.g. radio velocity under one standard of rest to frequency
  under another) and a reproducible, serialisable coordinate Mapping is required.
when_to_use_negative:
- The two SpecFrames already share the same System, Unit, and StdOfRest attributes,
  so no coordinate conversion Mapping is needed
- The task is a general WCS/FITS-WCS interpretation rather than a conversion between
  two explicitly defined spectral coordinate systems
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Starlink AST
  role: Provides the SpecFrame class and the convert method that generates the Mapping
    between the two spectral coordinate systems, returned as a FrameSet
  repo: https://github.com/Starlink/starlink
- name: Starlink AST library
  role: Generalised scheme for modelling, manipulating and storing inter-related coordinate
    systems underlying the SpecFrame convert operation
- name: Python
  role: Language used via the starlink.Ast bindings to create and deep-copy the SpecFrames,
    invoke convert, and serialise through an AST Channel
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:b43216559ff2823bf8558948a7bbdecf46f22e8c2a97d170372f55afbf68c6db
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - Python
  - Starlink AST
  - Starlink AST library
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# velocity-frequency-mapping-construction

## Summary

Constructs an AST Mapping between two different spectral coordinate systems — e.g. heliocentric radio velocity (System=VRAD, RestFreq=345.8 GHz) and LSR frequency (System=FREQ, Unit=Hz, StdOfRest=LSR) — by invoking the convert method on one SpecFrame with another as argument. The returned Mapping is encapsulated in a FrameSet that also holds copies of both Frames.

## When to use

Use when spectral data must be transformed between incompatible spectral coordinate systems (e.g. radio velocity under one standard of rest to frequency under another) and a reproducible, serialisable coordinate Mapping is required.

## When NOT to use

- The two SpecFrames already share the same System, Unit, and StdOfRest attributes, so no coordinate conversion Mapping is needed
- The task is a general WCS/FITS-WCS interpretation rather than a conversion between two explicitly defined spectral coordinate systems

## Inputs

- SpecFrame configured with System=VRAD and RestFreq=345.8 GHz (source frame)
- Deep-copied SpecFrame modified to System=FREQ, Unit=Hz, StdOfRest=LSR (target frame)

## Outputs

- FrameSet whose base and current frames are linked by the velocity-to-frequency Mapping and which includes copies of both SpecFrames
- Serialised AST text file (frameset_vrad_to_freq.ast) written via an AST Channel

## How to apply

Import the Starlink AST library through its Python bindings (starlink.Ast) and create a SpecFrame configured with System=VRAD and RestFreq=345.8 GHz. Deep-copy that SpecFrame and modify the copy to System=FREQ with Unit=Hz and StdOfRest=LSR. Call the convert method on the VRAD frame, passing the FREQ frame as argument: if a conversion is possible, AST generates a Mapping between the two spectral coordinate systems and returns it encapsulated within a FrameSet that also includes copies of both Frames. Serialise the FrameSet through an AST Channel and write the text output to a .ast file so it can be re-read later. Validate by re-reading the serialisation to recreate an equivalent FrameSet and confirming the base and current frames report System=VRAD and System=FREQ respectively.

## Related tools

- **Starlink AST** (Provides the SpecFrame class and the convert method that generates the Mapping between the two spectral coordinate systems, returned as a FrameSet) — https://github.com/Starlink/starlink
- **Starlink AST library** (Generalised scheme for modelling, manipulating and storing inter-related coordinate systems underlying the SpecFrame convert operation)
- **Python** (Language used via the starlink.Ast bindings to create and deep-copy the SpecFrames, invoke convert, and serialise through an AST Channel)

## Examples

```
import copy; from starlink import Ast; f1 = Ast.SpecFrame(System='VRAD', RestFreq=Ast.Units('345.8GHz')); f2 = copy.deepcopy(f1); f2.System = 'FREQ'; f2.Unit = 'Hz'; f2.StdOfRest = 'LSR'; fs = f1.convert(f2)
```

## Evaluation signals

- The serialisation file frameset_vrad_to_freq.ast exists after running the Channel write
- The .ast file can be re-read by AST to recreate an equivalent FrameSet
- The re-read FrameSet's base frame reports System=VRAD and its current frame reports System=FREQ
- The convert call returned non-None (a Mapping was possible between the two spectral systems)
- The returned FrameSet contains copies of both the original VRAD SpecFrame and the FREQ SpecFrame

## Limitations

- A Mapping is only generated 'if possible' — if no conversion path exists between the two spectral systems, convert will not yield a usable FrameSet
- The conversion correctness depends on correctly setting the rest frequency (RestFreq=345.8 GHz) and standard of rest (StdOfRest) attributes on the source and target SpecFrames before calling convert

## Evidence

- [other] convert generates Mapping between spectral coordinate systems: [evidence span withheld — anchor status: not_found]
- [other] FrameSet encapsulation of Mapping and both Frames: [evidence span withheld — anchor status: not_found]
