---
name: spectral-coordinate-system-conversion
description: Use when a spectrum's axis is expressed in one spectral system (System,
  Unit, StdOfRest, e.g. VRAD km/s heliocentric with RestFreq 345.8 GHz) and it must
  be re-expressed in another (e.g. FREQ in Hz, LSR standard of rest).
when_to_use_negative:
- The axis to transform is not a spectral coordinate (e.g. sky pixel or time axis)
  — SpecFrame conversions only relate spectral systems
- No rest frequency / standard-of-rest information is available, so AST cannot build
  the required velocity-to-frequency Mapping
- The target format is a plain FITS-WCS keyword update rather than an inter-related
  coordinate system; consider AST's FITS-WCS reading/writing facilities instead
edam_operation: http://edamontology.org/operation_3435
tools:
- name: Starlink AST
  role: Provides SpecFrame, the convert method, FrameSet and Channel objects used
    to model, transform and store the spectral coordinate systems
  repo: Starlink/starlink
- name: Python
  role: Language binding (starlink.Ast) through which the SpecFrame convert workflow
    is invoked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:449707f2b58f982299f04629854a29a4b0b673ba90682ab17e225d48e43651b1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - Python
  - Starlink AST
  edam_operation: http://edamontology.org/operation_3435
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectral-coordinate-system-conversion

## Summary

Use the Starlink AST library's SpecFrame convert method to generate a Mapping between two spectral coordinate systems (e.g. heliocentric radio velocity vs. LSR frequency), returned as a FrameSet that encapsulates both Frames. This replaces hand-coded velocity/frequency conversion formulae with AST's data-model-driven transformation.

## When to use

Use when a spectrum's axis is expressed in one spectral system (System, Unit, StdOfRest, e.g. VRAD km/s heliocentric with RestFreq 345.8 GHz) and it must be re-expressed in another (e.g. FREQ in Hz, LSR standard of rest).

## When NOT to use

- The axis to transform is not a spectral coordinate (e.g. sky pixel or time axis) — SpecFrame conversions only relate spectral systems
- No rest frequency / standard-of-rest information is available, so AST cannot build the required velocity-to-frequency Mapping
- The target format is a plain FITS-WCS keyword update rather than an inter-related coordinate system; consider AST's FITS-WCS reading/writing facilities instead

## Inputs

- SpecFrame configured with System=VRAD, Unit=km/s (or native unit), StdOfRest=HELIOCENTRIC, RestFreq=345.8 GHz
- Deep-copied SpecFrame retargeted to System=FREQ, Unit=Hz, StdOfRest=LSR
- Spectral axis values in the source system (e.g. radio velocities) to be mapped

## Outputs

- FrameSet whose base frame (System=VRAD) and current frame (System=FREQ) are connected by the conversion Mapping
- Serialised FrameSet text file (e.g. frameset_vrad_to_freq.ast) written via an AST Channel
- Converted spectral coordinate values (frequencies in Hz) obtained by evaluating the Mapping

## How to apply

Import the Starlink AST library through its Python bindings (starlink.Ast) and create a SpecFrame configured with the source attributes (System=VRAD, RestFreq=345.8 GHz). Deep-copy that SpecFrame and modify the copy to the target attributes (System=FREQ, Unit=Hz, StdOfRest=LSR). Invoke the convert method on the source frame with the target frame as argument; if the conversion is possible AST returns a FrameSet whose base and current frames are linked by the velocity-to-frequency Mapping, along with copies of both Frames. Serialise the FrameSet through an AST Channel to a text file (e.g. frameset_vrad_to_freq.ast) so it can be stored and re-read; AST's Channel provides the storage mechanism for inter-related coordinate systems described in the paper. Validate by re-reading the serialisation into an equivalent FrameSet and checking the base/current frames report System=VRAD and System=FREQ respectively.

## Related tools

- **Starlink AST** (Provides SpecFrame, the convert method, FrameSet and Channel objects used to model, transform and store the spectral coordinate systems) — Starlink/starlink
- **Python** (Language binding (starlink.Ast) through which the SpecFrame convert workflow is invoked)

## Examples

```
from starlink import Ast; f1=Ast.SpecFrame(System="VRAD",RestFreq=345.8e9); f2=Ast.SpecFrame(System="FREQ",Unit="Hz",StdOfRest="LSR"); fs=f1.convert(f2)
```

## Evaluation signals

- The returned object is a FrameSet whose base frame reports System=VRAD and whose current frame reports System=FREQ
- The serialisation file (frameset_vrad_to_freq.ast) exists and can be re-read by AST to recreate an equivalent FrameSet
- Mapping a test velocity through the FrameSet yields a frequency consistent with the stated RestFreq of 345.8 GHz (e.g. v=0 maps to the rest frequency)
- The FrameSet contains copies of both input Frames with their StdOfRest attributes preserved (heliocentric source, LSR target)
- AST reports no error on convert — the method returns None/raises only when the requested spectral conversion is not possible

## Limitations

- The convert method generates a Mapping only 'if possible' — conversions requiring missing attributes (e.g. an undefined RestFreq when moving between velocity and frequency systems) will fail
- Using the Python/Java bindings requires building or installing the underlying C library; the classic Starlink build requires the patched bootstrap chain (./bootstrap, make configure-deps, ./configure -C, make world) rather than stock autoconf versions
- Java-based components (starjava, e.g. JNIAST) are built separately via ant and a JDK 8+ and are not produced by the classic build procedure

## Evidence
