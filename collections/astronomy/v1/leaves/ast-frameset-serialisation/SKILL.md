---
name: ast-frameset-serialisation
description: Use when an AST FrameSet encoding inter-related coordinate systems (pixel↔FITS
  primary WCS, or VRAD↔FREQ spectral systems) must be persisted to or recovered from
  a text representation.
when_to_use_negative:
- The coordinate information is already in FITS-WCS header cards and you only need
  to read it — use a FitsChan read, not a Channel serialisation, at that step
- The object is a plain Mapping or single Frame with no inter-related coordinate systems
  to store — the FrameSet serialisation is designed for related Frames
tools:
- name: Starlink AST
  role: Provides the FrameSet, FitsChan, SpecFrame and Channel objects used to build,
    convert and serialise the coordinate-system representation
  repo: Starlink/starlink
- name: Python
  role: Host language for driving AST via the starlink.Ast bindings to construct Frames,
    call convert, and write Channel output
  repo: Starlink/starlink
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:4355761e2697db60e27f4865e9b3ec671c3e966cd24043e50c81e8defafa4943
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Starlink/starlink
  tools:
  - C
  - Python
  - Starlink AST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ast-frameset-serialisation

## Summary

Serialise an AST FrameSet — e.g. one produced by a FitsChan destructive read of FITS-WCS headers or by a SpecFrame convert call — through an AST Channel to a text file that can be re-read to recreate an equivalent FrameSet.

## When to use

Use when an AST FrameSet encoding inter-related coordinate systems (pixel↔FITS primary WCS, or VRAD↔FREQ spectral systems) must be persisted to or recovered from a text representation.

## When NOT to use

- The coordinate information is already in FITS-WCS header cards and you only need to read it — use a FitsChan read, not a Channel serialisation, at that step
- The object is a plain Mapping or single Frame with no inter-related coordinate systems to store — the FrameSet serialisation is designed for related Frames

## Inputs

- AST FrameSet from a FitsChan destructive read (astRead) of FITS-WCS header cards
- AST FrameSet returned by Frame.convert between two SpecFrames (e.g. System=VRAD vs System=FREQ)
- Residual FitsChan header buffer remaining after the destructive read

## Outputs

- Text serialisation of the FrameSet written via an AST Channel (e.g. frameset_vrad_to_freq.ast)
- Text file of the residual header buffer (WCS cards removed by the destructive read)

## How to apply

After obtaining a FrameSet — for example from a destructive astRead on a FitsChan holding FITS-WCS cards (CTYPE/CRVAL/CRPIX-type), whose base Frame is pixel coordinates and current Frame is the FITS primary WCS frame, or from frame1.convert(frame2) between two SpecFrames with differing System, Unit and StdOfRest attributes — pass the FrameSet through an AST Channel and write the resulting text output to a .ast file. The Channel serialisation captures the FrameSet including its base/current Frame assignments and any Mapping between Frames (e.g. a velocity-to-frequency Mapping between System=VRAD, RestFreq=345.8 GHz and System=FREQ, Unit=Hz, StdOfRest=LSR). For the FITS route, also write the residual FitsChan buffer to a second file to document which WCS cards the destructive read consumed. Validation consists of re-reading the serialised file with AST and confirming it recreates an equivalent FrameSet with the expected base/current Frame System values.

## Related tools

- **Starlink AST** (Provides the FrameSet, FitsChan, SpecFrame and Channel objects used to build, convert and serialise the coordinate-system representation) — Starlink/starlink
- **Python** (Host language for driving AST via the starlink.Ast bindings to construct Frames, call convert, and write Channel output) — Starlink/starlink

## Examples

```
import starlink.Ast as Ast; fs = frame_vrad.convert(frame_freq); Ast.write(fs, 'frameset_vrad_to_freq.ast')
```

## Evaluation signals

- Re-reading the serialised .ast file through AST recreates an equivalent FrameSet
- The FrameSet from a FITS-WCS read has base Frame = pixel coordinates and current Frame = FITS primary WCS
- The converted SpecFrame FrameSet reports base/current frames with System=VRAD and System=FREQ respectively
- The residual FitsChan buffer contains none of the WCS cards (CTYPE/CRVAL/CRPIX-type) supplied on input, confirming the read was destructive
- The serialisation output file exists and is parseable by AST

## Limitations

- The FitsChan Read method is destructive: the WCS cards are deleted from the buffer, so the original header state is not preserved unless the residual buffer is written out
- Frame.convert generates a Mapping between two spectral systems only 'if possible' — incompatible System/Unit/StdOfRest combinations may yield no Mapping

## Evidence

- [abstract] AST provides facilities for attaching world co-ordinate systems to astronomical data and retrieving/interpreting that information in formats including FITS-WCS: "AST provides a comprehensive range of facilities for attaching world co-ordinate systems to astronomical data, and for retrieving and interpreting that information in a variety of formats, including"
- [intro] The Starlink AST library provides a generalised scheme for modelling, manipulating and storing inter-related coordinate systems.: "The Starlink AST library (Warren-Smith and Berry, 2013, ascl:1404.016) provides a generalised scheme for modelling, manipulating and storing inter-related coordinate systems."
- [intro] AST is written in C but has Python bindings, enabling the serialisation workflow to be scripted.: "Whilst written in C, it has bindings for several other languages including Python, Java, Perl and Fortran."
