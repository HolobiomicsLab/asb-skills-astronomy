---
name: frameset-fits-serialization
description: Use when a coordinate conversion Mapping between Frames (e.g. ICRS to
  Galactic SkyFrames) must be persisted to disk and later re-loaded and validated
  through AST's FrameSet FITS channels.
when_to_use_negative:
- The Mapping between the two Frames does not exist — convert() will not yield a usable
  FrameSet and serialization is pointless.
- The target format is not FITS (e.g. native AST text or other WCS conventions like
  FITS-WCS read directly), where a different AST channel should be used.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: Starlink AST
  role: Library whose SkyFrame, convert method, FrameSet and FITS write channel implement
    the coordinate conversion and its serialization
  repo: https://github.com/Starlink/starlink
- name: Starlink AST library
  role: Provides the generalised scheme for modelling, manipulating and storing inter-related
    coordinate systems used by this skill
  repo: https://github.com/Starlink/starlink
- name: Python
  role: Language binding (starlink.Ast) used to instantiate SkyFrames, call convert(),
    and drive the FITS write/read channels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:ac2c679cef098a1ea581e43048354208581707ce46456b1cce666e06a6fe5c86
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - Python
  - Starlink AST
  - Starlink AST library
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# frameset-fits-serialization

## Summary

Serialize an AST FrameSet — e.g. the Mapping between an ICRS (RA,Dec) SkyFrame and a Galactic (l,b) SkyFrame produced by the convert method — to disk as a FITS file (frameset.fits) via the FrameSet FITS write channel, then re-load it to verify the Mapping still transforms coordinates correctly.

## When to use

Use when a coordinate conversion Mapping between Frames (e.g. ICRS to Galactic SkyFrames) must be persisted to disk and later re-loaded and validated through AST's FrameSet FITS channels.

## When NOT to use

- The Mapping between the two Frames does not exist — convert() will not yield a usable FrameSet and serialization is pointless.
- The target format is not FITS (e.g. native AST text or other WCS conventions like FITS-WCS read directly), where a different AST channel should be used.

## Inputs

- AST SkyFrame object (default ICRS RA/Dec System)
- Deep-copied SkyFrame with System attribute set to 'Galactic'
- FrameSet returned by SkyFrame.convert()
- Test ICRS (RA,Dec) coordinate pair

## Outputs

- frameset.fits — FrameSet serialized via the FITS write channel
- Re-loaded AST FrameSet parsed back from frameset.fits
- Finite, in-range Galactic (l,b) values transformed from a test (RA,Dec) input

## How to apply

Instantiate a SkyFrame (default System is ICRS RA/Dec), deep-copy it and set the copy's System attribute to 'Galactic' to represent Galactic longitude and latitude. Call frame1.convert(frame2) with an empty domain/spec string; if a Mapping is possible it is returned encapsulated within a FrameSet that also contains copies of frame1 and frame2. Write this FrameSet to disk as frameset.fits using the FrameSet FITS write channel. Re-load frameset.fits, select the Mapping between the base (ICRS RA,Dec) and current (Galactic l,b) Frames, and push a test (RA,Dec) pair through it, checking the returned (l,b) values are finite and in-range.

## Related tools

- **Starlink AST** (Library whose SkyFrame, convert method, FrameSet and FITS write channel implement the coordinate conversion and its serialization) — https://github.com/Starlink/starlink
- **Starlink AST library** (Provides the generalised scheme for modelling, manipulating and storing inter-related coordinate systems used by this skill) — https://github.com/Starlink/starlink
- **Python** (Language binding (starlink.Ast) used to instantiate SkyFrames, call convert(), and drive the FITS write/read channels)

## Examples

```
from starlink import Ast; f1 = Ast.SkyFrame(); f2 = Ast.SkyFrame(); f2.System = 'Galactic'; fs = f1.convert(f2, ''); fs.write('frameset.fits')
```

## Evaluation signals

- frameset.fits exists on disk after the write step
- The saved file parses back as a valid AST FrameSet
- The Mapping between base (ICRS) and current (Galactic) Frames is selectable from the re-loaded FrameSet
- A test (RA,Dec) pair pushed through the Mapping yields finite Galactic (l,b) values
- The resulting (l,b) outputs are in the expected Galactic longitude/latitude range

## Limitations

- convert() only generates a Mapping between the two Frames 'if possible'; unrelated or incompatible Frames yield no usable FrameSet.
- The FrameSet returned by convert() contains copies of frame1 and frame2, so subsequent attribute changes to the originals do not affect the serialized Mapping.
- The skill relies on AST's own FrameSet FITS channel; arbitrary third-party FITS readers may not reconstruct the full Mapping semantics.

## Evidence

- [other] Calling frame1.convert(frame2) on the two SkyFrames generates, if possible, a Mapping from frame1 to frame2, and this Mapping is returned encapsulated within a FrameSet: [evidence span withheld — anchor status: not_found]
- [other] Serialise the returned FrameSet to disk as frameset.fits using the FrameSet FITS write channel.: [evidence span withheld — anchor status: not_found]
