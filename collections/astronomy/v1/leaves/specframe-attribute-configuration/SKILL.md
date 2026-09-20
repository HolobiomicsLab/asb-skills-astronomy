---
name: specframe-attribute-configuration
description: 'Use when a spectral coordinate must be re-expressed in another system
  (e.g. heliocentric VRAD at RestFreq=345.8 GHz to LSR frequency in Hz): set up two
  SpecFrames with differing System/Unit/StdOfRest and convert between them.'
when_to_use_negative:
- The coordinate systems of interest are not spectral (e.g. sky or time frames), where
  a plain Frame rather than a SpecFrame with RestFreq/StdOfRest is the appropriate
  object
- Both frames already share identical System, Unit and StdOfRest attributes, so no
  inter-system Mapping is required and convert has nothing to compute
- You need a standalone velocity-to-frequency formula rather than a Mapping embedded
  in a FrameSet structure — the AST convert route always returns the Mapping inside
  a FrameSet containing copies of both Frames
tools:
- name: Starlink AST
  role: Provides the SpecFrame class, its configurable System/Unit/RestFreq/StdOfRest
    attributes, the convert method that generates the inter-system Mapping, and the
    Channel used for serialisation
  repo: https://github.com/Starlink/starlink
- name: Starlink AST library
  role: The underlying generalised scheme for modelling, manipulating and storing
    inter-related coordinate systems in which SpecFrames and FrameSets live
  repo: https://github.com/Starlink/starlink
- name: Python
  role: Binding language used to instantiate the SpecFrames, call convert, and write
    the Channel output (starlink.Ast)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.02.003
    title: 'AST: A library for modelling and manipulating coordinate systems'
  source_content_hash: sha256:6cee2e0b9bf45548deffaf5bb70868305fede176f7aea16e72afedf328254297
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# specframe-attribute-configuration

## Summary

Configures AST SpecFrame attributes (System, Unit, RestFreq, StdOfRest) so that two different spectral coordinate systems — e.g. heliocentric radio velocity versus LSR frequency — can be related by AST's Convert method, which returns a FrameSet encapsulating the Mapping between them.

## When to use

Use when a spectral coordinate must be re-expressed in another system (e.g. heliocentric VRAD at RestFreq=345.8 GHz to LSR frequency in Hz): set up two SpecFrames with differing System/Unit/StdOfRest and convert between them.

## When NOT to use

- The coordinate systems of interest are not spectral (e.g. sky or time frames), where a plain Frame rather than a SpecFrame with RestFreq/StdOfRest is the appropriate object
- Both frames already share identical System, Unit and StdOfRest attributes, so no inter-system Mapping is required and convert has nothing to compute
- You need a standalone velocity-to-frequency formula rather than a Mapping embedded in a FrameSet structure — the AST convert route always returns the Mapping inside a FrameSet containing copies of both Frames

## Inputs

- A SpecFrame configured with System=VRAD and RestFreq=345.8 GHz (source spectral system)
- A deep copy of that SpecFrame reconfigured to System=FREQ, Unit=Hz, StdOfRest=LSR (target spectral system)

## Outputs

- A FrameSet whose base and current frames report System=VRAD and System=FREQ respectively, encapsulating the velocity-to-frequency Mapping
- A serialised AST text file (e.g. frameset_vrad_to_freq.ast) produced via an AST Channel

## How to apply

Import the Starlink AST library through its Python bindings (starlink.Ast) and create a SpecFrame configured with System=VRAD and RestFreq=345.8 GHz for the heliocentric radio-velocity system. Deep-copy that SpecFrame and modify the copy's attributes to System=FREQ, Unit=Hz and StdOfRest=LSR, so the two frames differ only in their spectral representation. Invoke the convert method on the VRAD frame with the FREQ frame as argument; if the conversion is possible, AST returns a FrameSet whose base and current frames are linked by the velocity-to-frequency Mapping and which also contains copies of both Frames. Serialise the resulting FrameSet through an AST Channel to persist the mapping (e.g. to frameset_vrad_to_freq.ast) so it can be re-read later.

## Related tools

- **Starlink AST** (Provides the SpecFrame class, its configurable System/Unit/RestFreq/StdOfRest attributes, the convert method that generates the inter-system Mapping, and the Channel used for serialisation) — https://github.com/Starlink/starlink
- **Starlink AST library** (The underlying generalised scheme for modelling, manipulating and storing inter-related coordinate systems in which SpecFrames and FrameSets live) — https://github.com/Starlink/starlink
- **Python** (Binding language used to instantiate the SpecFrames, call convert, and write the Channel output (starlink.Ast))

## Examples

```
import starlink.Ast as Ast; f1=Ast.SpecFrame(System='VRAD',RestFreq='345.8 GHz'); f2=f1.copy(); f2.System='FREQ'; f2.Unit='Hz'; f2.StdOfRest='LSR'; fs=f1.convert(f2)
```

## Evaluation signals

- The serialisation file frameset_vrad_to_freq.ast exists and can be re-read by AST to recreate an equivalent FrameSet
- Re-reading the serialised FrameSet and querying its frames confirms the base frame reports System=VRAD and the current frame reports System=FREQ
- The FrameSet contains copies of both source SpecFrames in addition to the Mapping between them
- The convert call succeeds (a Mapping is generated) only because the two frames are describable spectral systems — a mismatched RestFreq or unsupported StdOfRest pairing should be checked to fail or return no Mapping

## Limitations

- The convert method only generates a Mapping 'if possible' — conversions between arbitrary spectral systems may not be achievable
- The returned Mapping is always encapsulated within a FrameSet that also includes copies of both Frames, not delivered as a bare Mapping
- Building Starlink classic software from source requires a specific bootstrap sequence (./bootstrap, make configure-deps, ./configure -C, make world), and the patched autoconf/automake/libtool versions shipped with it must be used, so the Python bindings may not be trivially available

## Evidence

- [full_text] Calling Convert on one SpecFrame with another as argument generates, if possible, a Mapping between the two spectral coordinate systems.: [evidence span withheld — anchor status: not_found]
- [full_text] The returned Mapping is encapsulated within a Frame Set that also includes copies of both Frames.: [evidence span withheld — anchor status: not_found]
