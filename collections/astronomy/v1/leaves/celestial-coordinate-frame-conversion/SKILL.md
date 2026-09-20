---
name: celestial-coordinate-frame-conversion
description: Use when a task needs to transform astronomical positions between celestial
  frames — e.g. converting ICRS (RA,Dec) coordinates to Galactic (l,b) — via the Starlink
  AST Python bindings.
when_to_use_negative:
- The coordinate metadata is already FITS-WCS attached to image data and only needs
  retrieval/interpretation rather than inter-frame Mapping generation
- The task requires conversion between non-celestial systems (e.g. spectral or time
  axes) not represented as SkyFrames
tools:
- name: Starlink AST
  role: Library whose SkyFrame and convert methods build the ICRS-to-Galactic Mapping
    and FrameSet
- name: Starlink AST library
  role: Generalised scheme for modelling, manipulating and storing inter-related coordinate
    systems used by the skill
- name: Python
  role: Language binding through which SkyFrames are instantiated and convert is invoked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:c1e9f730c149dff1f630ef4008323763534494d0c6c3f91cde72ad636cd6b41c
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
yes_rate: 1.0
n_claims: 10
---

# celestial-coordinate-frame-conversion

## Summary

Generates a Mapping between two celestial coordinate systems (e.g. ICRS RA/Dec and Galactic l/b) using the convert method on Starlink AST SkyFrames, returned encapsulated in a FrameSet that can be serialised to FITS and re-loaded for transformation of test coordinates.

## When to use

Use when a task needs to transform astronomical positions between celestial frames — e.g. converting ICRS (RA,Dec) coordinates to Galactic (l,b) — via the Starlink AST Python bindings.

## When NOT to use

- The coordinate metadata is already FITS-WCS attached to image data and only needs retrieval/interpretation rather than inter-frame Mapping generation
- The task requires conversion between non-celestial systems (e.g. spectral or time axes) not represented as SkyFrames

## Inputs

- Starlink AST SkyFrame object (default ICRS RA/Dec attributes)
- Target SkyFrame copy with System attribute set to 'Galactic'
- Empty domain/spec string passed to convert
- A test (RA,Dec) coordinate pair for validation

## Outputs

- FrameSet encapsulating the ICRS-to-Galactic Mapping plus copies of both SkyFrames
- frameset.fits — the FrameSet serialised through a FITS write channel
- Finite, in-range Galactic (l,b) values produced by transforming the test (RA,Dec) pair

## How to apply

Instantiate a SkyFrame with the Starlink.Ast Python bindings; by default it carries ICRS right ascension and declination attributes. Deep-copy that frame and set the copy's System attribute to 'Galactic' so it represents Galactic longitude and latitude. Call convert on the original frame with the Galactic copy as target (and an empty domain/spec string); if possible this generates a Mapping from frame1 to frame2, returned encapsulated within a FrameSet that also includes copies of frame1 and frame2. Serialise the FrameSet to disk as frameset.fits via the FrameSet FITS write channel, then re-load it, select the Mapping between the base (ICRS RA,Dec) and current (Galactic l,b) Frames, and push a test (RA,Dec) pair through it. The rationale is that AST's data model stores inter-related coordinate systems so the conversion Mapping persists and can be validated after a round-trip through FITS.

## Related tools

- **Starlink AST** (Library whose SkyFrame and convert methods build the ICRS-to-Galactic Mapping and FrameSet)
- **Starlink AST library** (Generalised scheme for modelling, manipulating and storing inter-related coordinate systems used by the skill)
- **Python** (Language binding through which SkyFrames are instantiated and convert is invoked)

## Examples

```
from starlink.Ast import SkyFrame; icrs = SkyFrame(); gal = icrs.copy(); gal.System = 'Galactic'; fs = icrs.convert(gal, '')
```

## Evaluation signals

- The saved frameset.fits exists and parses back as an AST FrameSet
- The Mapping selected between the base (ICRS RA,Dec) and current (Galactic l,b) Frames transforms a test ICRS coordinate into finite Galactic values
- The transformed (l,b) outputs fall within valid Galactic longitude/latitude ranges
- The returned FrameSet contains copies of both the source and target SkyFrames

## Limitations

- convert only generates a Mapping between the two SkyFrames 'if possible' — no Mapping is produced for systems it cannot relate
- The article describes AST as written in C with bindings for Python, Java, Perl and Fortran, so availability depends on the installed binding

## Evidence
