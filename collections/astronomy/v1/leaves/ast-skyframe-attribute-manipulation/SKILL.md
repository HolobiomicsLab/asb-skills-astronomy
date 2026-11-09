---
name: ast-skyframe-attribute-manipulation
description: Use when a task requires converting sky coordinates between systems (e.g.
  ICRS RA,Dec to Galactic l,b) using AST, and the conversion should be obtained as
  a reusable, serialisable FrameSet Mapping.
when_to_use_negative:
- Input coordinates are not sky coordinates on a celestial sphere (e.g. pixel or spectral
  axes) — a SkyFrame/convert pattern between SkyFrames is the wrong frame pair
- The task needs a new Mapping invented from scratch rather than a conversion between
  two Frames that AST can connect via its built-in frame relations
tools:
- name: Starlink AST
  role: Provides the SkyFrame class, its System attribute, the convert method, FrameSet,
    and FITS read/write channels used to generate and serialise the ICRS-to-Galactic
    Mapping
  repo: https://github.com/Starlink/starlink
- name: Python
  role: Language binding through which the AST data model (SkyFrame, FrameSet, convert)
    is invoked in this workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:de43da28e6b9069b3d5e3308d1c6b02560d6fa854acf875585d247fb35a74e27
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - Python
  - Starlink AST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ast-skyframe-attribute-manipulation

## Summary

Uses the Starlink AST library's SkyFrame class to build a coordinate-system Mapping — e.g. ICRS (RA,Dec) to Galactic (l,b) — by deep-copying a SkyFrame, setting its System attribute, and invoking the convert method, which returns the Mapping encapsulated in a FrameSet.

## When to use

Use when a task requires converting sky coordinates between systems (e.g. ICRS RA,Dec to Galactic l,b) using AST, and the conversion should be obtained as a reusable, serialisable FrameSet Mapping.

## When NOT to use

- Input coordinates are not sky coordinates on a celestial sphere (e.g. pixel or spectral axes) — a SkyFrame/convert pattern between SkyFrames is the wrong frame pair
- The task needs a new Mapping invented from scratch rather than a conversion between two Frames that AST can connect via its built-in frame relations

## Inputs

- An AST SkyFrame instance (default ICRS RA,Dec attributes)
- A deep copy of the SkyFrame with its System attribute set to 'Galactic' (l,b)
- A target spec/domain string passed to convert (empty string is sufficient here)
- A test ICRS (RA,Dec) coordinate pair for validating the Mapping

## Outputs

- An AST FrameSet encapsulating the ICRS-to-Galactic Mapping together with copies of frame1 and frame2
- A serialised frameset.fits FITS file that parses back as an AST FrameSet
- Finite, in-range Galactic (l,b) values transformed from the test (RA,Dec) pair

## How to apply

Instantiate a SkyFrame via the starlink.Ast Python bindings — by default it carries ICRS right ascension and declination attributes. Deep-copy that frame and set the copy's System attribute to 'Galactic' so it represents Galactic longitude and latitude. Call the original frame's convert method with the Galactic copy as target (and an empty domain/spec string); if possible, AST generates a Mapping from frame1 to frame2 returned inside a FrameSet that also includes copies of frame1 and frame2. Serialise the FrameSet (e.g. to frameset.fits via a FITS write channel), reload it, select the Mapping between the base (ICRS) and current (Galactic) Frames, and push a test coordinate through it. The deep-copy plus attribute-edit pattern is what makes the Mapping represent a different SkyFrame System rather than an identical frame.

## Related tools

- **Starlink AST** (Provides the SkyFrame class, its System attribute, the convert method, FrameSet, and FITS read/write channels used to generate and serialise the ICRS-to-Galactic Mapping) — https://github.com/Starlink/starlink
- **Python** (Language binding through which the AST data model (SkyFrame, FrameSet, convert) is invoked in this workflow)

## Examples

```
from starlink import Ast; import copy; sf = Ast.SkyFrame(); gal = copy.deepcopy(sf); gal.System = 'Galactic'; fs = sf.convert(gal, ""); fs.write('frameset.fits')
```

## Evaluation signals

- frameset.fits exists on disk and parses back as an AST FrameSet
- The Mapping extracted between base (ICRS RA,Dec) and current (Galactic l,b) Frames is not None (i.e. convert succeeded)
- A test (RA,Dec) pair pushed through the Mapping yields finite Galactic (l,b) values
- The transformed (l,b) values are in range (longitude within 0–360 deg or ±180 deg, latitude within ±90 deg)

## Limitations

- convert only generates a Mapping 'if possible' — frame pairs with no route through AST's frame relations yield no usable FrameSet
- The returned FrameSet also contains copies of frame1 and frame2, so care is needed when selecting the intended Mapping between base and current Frames
- Building the underlying library from source requires the specific Starlink build sequence (./bootstrap, make configure-deps, ./configure -C, make world), which is a nontrivial prerequisite

## Evidence
