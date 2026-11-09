---
name: icrs-to-galactic-mapping-verification
description: Use when an analysis needs a validated celestial coordinate transformation
  between ICRS (RA,Dec) and Galactic (l,b) frames and requires proof that the AST-generated
  Mapping round-trips through FITS and yields finite, in-range outputs.
when_to_use_negative:
- The input coordinates are already in the same frame (no ICRS-to-Galactic transformation
  is needed)
- The frames are not SkyFrames or the requested frame pair cannot be connected by
  AST, in which case convert generates no Mapping ('if possible' condition fails)
  rather than a usable FrameSet
tools:
- name: Starlink AST
  role: Provides the data model (SkyFrame, Mapping, FrameSet) and the convert method
    used to generate and encapsulate the ICRS-to-Galactic coordinate transformation
- name: Starlink AST library
  role: Underlying generalised scheme for modelling, manipulating and storing inter-related
    coordinate systems; installed AST library whose Python bindings are driven
- name: Python
  role: Language binding used to instantiate SkyFrames, call frame1.convert(frame2),
    and serialise/reload frameset.fits
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:d2a72d0492ef68b5d7eae0d740fef6743248d798f45c6d330041764c1dfe75ce
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# icrs-to-galactic-mapping-verification

## Summary

Verifies that the Starlink AST library can build a usable ICRS (RA,Dec) to Galactic (l,b) coordinate conversion by creating two SkyFrames and using the convert method, which returns the Mapping encapsulated in a FrameSet that is serialised to FITS and round-trip tested.

## When to use

Use when an analysis needs a validated celestial coordinate transformation between ICRS (RA,Dec) and Galactic (l,b) frames and requires proof that the AST-generated Mapping round-trips through FITS and yields finite, in-range outputs.

## When NOT to use

- The input coordinates are already in the same frame (no ICRS-to-Galactic transformation is needed)
- The frames are not SkyFrames or the requested frame pair cannot be connected by AST, in which case convert generates no Mapping ('if possible' condition fails) rather than a usable FrameSet

## Inputs

- A default AST SkyFrame carrying ICRS right ascension and declination attributes (constructed via starlink.Ast Python bindings)
- A deep copy of that SkyFrame with its System attribute set to 'Galactic' (Galactic longitude and latitude)
- A test ICRS (RA,Dec) coordinate pair for round-trip validation

## Outputs

- An AST FrameSet encapsulating the ICRS-to-Galactic Mapping together with copies of the base and current SkyFrames
- A serialised frameset.fits file written via the FrameSet FITS write channel
- Finite, in-range Galactic (l,b) values transformed from the test ICRS (RA,Dec) pair

## How to apply

Using the starlink.Ast Python bindings of the installed AST library, instantiate a SkyFrame, which by default carries ICRS right ascension and declination attributes, then deep-copy it and set the copy's System attribute to 'Galactic' so it represents Galactic longitude and latitude. Invoke frame1.convert(frame2) with the Galactic copy as target and an empty domain/spec string; if possible AST generates a Mapping from frame1 to frame2 returned encapsulated within a FrameSet that also includes copies of both frames. Serialise the FrameSet to disk as frameset.fits via the FrameSet FITS write channel, then re-load it and select the Mapping between the base (ICRS RA,Dec) and current (Galactic l,b) Frames. Finally, push a test (RA,Dec) pair through the Mapping to confirm finite, in-range (l,b) outputs, demonstrating a usable coordinate conversion.

## Related tools

- **Starlink AST** (Provides the data model (SkyFrame, Mapping, FrameSet) and the convert method used to generate and encapsulate the ICRS-to-Galactic coordinate transformation)
- **Starlink AST library** (Underlying generalised scheme for modelling, manipulating and storing inter-related coordinate systems; installed AST library whose Python bindings are driven)
- **Python** (Language binding used to instantiate SkyFrames, call frame1.convert(frame2), and serialise/reload frameset.fits)

## Examples

```
from starlink import Ast; f1 = Ast.SkyFrame(); f2 = f1.copy(); f2.System = 'Galactic'; fs = f1.convert(f2, ''); fs.writefits('frameset.fits')
```

## Evaluation signals

- The saved frameset.fits file exists on disk
- frameset.fits parses back successfully as an AST FrameSet
- The Mapping selected between the base (ICRS RA,Dec) and current (Galactic l,b) Frames transforms a test (RA,Dec) pair without error
- The resulting Galactic (l,b) outputs are finite and in range
- The reloaded FrameSet includes copies of frame1 and frame2, confirming the Mapping is encapsulated with its source and target frames

## Limitations

- The convert method only generates a Mapping 'if possible'; for some frame pairs AST cannot connect them and no usable FrameSet is produced
- Validation is by a single test coordinate pair with finite/in-range outputs, not a systematic accuracy comparison against a reference implementation

## Evidence

- [other] Calling frame1.convert(frame2) on the two SkyFrames generates, if possible, a Mapping from frame1 to frame2: [evidence span withheld — anchor status: not_found]
