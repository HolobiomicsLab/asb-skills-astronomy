---
name: astronomical-world-coordinate-system-interpretation
description: Use when a buffer of FITS header cards containing WCS keywords (CTYPE/CRVAL/CRPIX-type
  cards) must be interpreted into an explicit coordinate-system model, i.e. an AST
  FrameSet whose base Frame is pixel coordinates and whose current Frame is the FITS
  primary WCS frame.
when_to_use_negative:
- The coordinate information is already available as an AST FrameSet or Channel text
  file — no FitsChan read is needed.
- The FITS header uses a non-FITS-WCS encoding (e.g. native/alternate encodings) without
  reconfiguring the FitsChan encoding, since the read interprets cards according to
  the FITS-WCS encoding.
- You need to preserve the original WCS header cards intact — the ast Read method
  is destructive and deletes the relevant cards from the buffer.
tools:
- name: Starlink AST
  role: Provides FitsChan and Channel classes used to ingest FITS-WCS cards, perform
    the destructive read into a FrameSet, and serialise the result
  repo: https://github.com/Starlink/starlink
- name: Python
  role: Language binding used to construct the FitsChan and call astRead on the FITS
    header buffer
  repo: https://github.com/Starlink/starlink
- name: C
  role: Native language of the AST library in which the FitsChan destructive read
    is implemented
  repo: https://github.com/Starlink/starlink
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:e8f80aae75ab7660b85db54b689ea7f1de631d7caca1bebe5e00249f7d6c2ead
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# astronomical-world-coordinate-system-interpretation

## Summary

Reconstructs world coordinate system information from FITS header cards into a Starlink AST FrameSet using the FitsChan destructive read (astRead), so that pixel coordinates and the FITS primary WCS frame are represented as connected Frames. This is the canonical AST route for interpreting FITS-WCS metadata attached to astronomical data.

## When to use

Use when a buffer of FITS header cards containing WCS keywords (CTYPE/CRVAL/CRPIX-type cards) must be interpreted into an explicit coordinate-system model, i.e. an AST FrameSet whose base Frame is pixel coordinates and whose current Frame is the FITS primary WCS frame.

## When NOT to use

- The coordinate information is already available as an AST FrameSet or Channel text file — no FitsChan read is needed.
- The FITS header uses a non-FITS-WCS encoding (e.g. native/alternate encodings) without reconfiguring the FitsChan encoding, since the read interprets cards according to the FITS-WCS encoding.
- You need to preserve the original WCS header cards intact — the ast Read method is destructive and deletes the relevant cards from the buffer.

## Inputs

- Buffer of FITS header cards encoding WCS information (CTYPE, CRVAL, CRPIX-type cards)
- FitsChan configured with the FITS-WCS encoding
- Starlink AST bindings from Python or C

## Outputs

- AST FrameSet with base Frame = pixel coordinates and current Frame = FITS primary WCS frame
- Residual FitsChan header buffer with WCS cards removed
- Serialised FrameSet text file written via an AST Channel
- Residual header buffer written to a second file

## How to apply

Construct an AST FitsChan (Starlink AST, called from Python via its bindings or from C) using the FITS-WCS encoding, and feed it the buffer of FITS header cards. Perform the destructive read (astRead) on the FitsChan: AST reads the FrameSet described by the cards and then deletes all the relevant WCS cards from the buffer, so this consumes CTYPE/CRVAL/CRPIX-type cards. Verify the returned FrameSet has its base Frame set to pixel coordinates and its current Frame set to the FITS primary WCS representation. Inspect the residual FitsChan buffer to confirm the consumed WCS cards were removed, since a successful read is by definition destructive. Finally serialise the FrameSet to a text file via an AST Channel and write the residual header buffer to a second file for inspection. This works because AST provides facilities for attaching world co-ordinate systems to astronomical data and retrieving/interpreting that information in formats including FITS-WCS.

## Related tools

- **Starlink AST** (Provides FitsChan and Channel classes used to ingest FITS-WCS cards, perform the destructive read into a FrameSet, and serialise the result) — https://github.com/Starlink/starlink
- **Python** (Language binding used to construct the FitsChan and call astRead on the FITS header buffer) — https://github.com/Starlink/starlink
- **C** (Native language of the AST library in which the FitsChan destructive read is implemented) — https://github.com/Starlink/starlink

## Evaluation signals

- The returned FrameSet has its base Frame set to pixel coordinates and its current Frame set to the FITS primary WCS frame.
- The residual FitsChan buffer contains none of the WCS cards supplied on input (CTYPE/CRVAL/CRPIX-type cards removed).
- The FrameSet serialises successfully through an AST Channel to a text file.
- The residual header buffer written to the second file contains no consumed WCS cards.
- The FitsChan was configured with the FITS-WCS encoding before the read.

## Limitations

- The ast Read method is destructive: it deletes all relevant FITS cards from the FitsChan buffer after reading, so the original header cards are not preserved unless saved separately.
- The FrameSet returned by astRead encodes the FITS primary WCS representation; interpretation depends on the FitsChan encoding being set to FITS-WCS.
- Interpreting WCS via AST requires the AST library and its language bindings; the library is written in C with bindings for Python, Java, Perl and Fortran.

## Evidence

- [discussion] The FitsChan's ast Read method performs a destructive read of the FITS-WCS cards into a FrameSet: [evidence span withheld — anchor status: not_found]
