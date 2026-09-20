---
name: fits-header-keyword-handling
description: Use when building an IXPE instrumental background template from pha1
  dark-field spectra and the per-file rates must be rescaled by livetime and BACKSCAL
  read from the FITS headers.
when_to_use_negative:
- The observation is dominated by solar flaring background, which can be persistently
  strongly polarized and is therefore not represented by a stable unpolarized template.
- The goal is background subtraction of already-binned Stokes products rather than
  constructing a new instrumental background template from dark-field spectra.
- The FITS headers lack valid BACKSCAL or LIVETIME keywords, so livetime/area normalization
  cannot be performed correctly.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ixpeobssim
  role: Provides the FITS utilities for reading pha1 headers (BACKSCAL, LIVETIME)
    and the create_background_template / xTemplateInstrumentalBkg machinery that consumes
    the resulting ascii template
  repo: https://github.com/lucabaldini/ixpeobssim
- name: Python
  role: Language in which the header-reading, rate normalization, averaging, and spline
    smoothing steps are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:2598e7b2463b99c249fa313557cce7880cd7065d79c78823f373fc5dcfc6f6ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - Python
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# FITS header keyword handling for IXPE background template generation

## Summary

Reads exposure-relevant keywords (BACKSCAL, LIVETIME) from PHA1 spectrum FITS headers with ixpeobssim FITS utilities so that dark-field background rates can be livetime- and area-normalized into a single instrumental background template. It underpins xTemplateInstrumentalBkg, which relies on the background being stable and unpolarized.

## When to use

Use when building an IXPE instrumental background template from pha1 dark-field spectra and the per-file rates must be rescaled by livetime and BACKSCAL read from the FITS headers.

## When NOT to use

- The observation is dominated by solar flaring background, which can be persistently strongly polarized and is therefore not represented by a stable unpolarized template.
- The goal is background subtraction of already-binned Stokes products rather than constructing a new instrumental background template from dark-field spectra.
- The FITS headers lack valid BACKSCAL or LIVETIME keywords, so livetime/area normalization cannot be performed correctly.

## Inputs

- pha1 spectrum FITS file(s) from a dark-field observation (with BACKSCAL and LIVETIME header keywords populated)

## Outputs

- background_template.txt: an ascii energy-spectrum table of smoothed, livetime- and area-normalized background rates suitable as input to an xTemplateInstrumentalBkg object

## How to apply

Load the input pha1 spectrum file(s) for the dark field using ixpeobssim's FITS utilities, reading the BACKSCAL keyword and the LIVETIME from each file's header rather than assuming defaults. For each file, extract the background rate, multiply it by the livetime, and normalize with respect to the total detector area; this rescaling by livetime/backscal is the same facility ixpeobssim provides for background subtraction of binned products. Average the per-file spectra and rescale back to physical units to form a single average background spectrum over the full IXPE energy band. Smooth the average with the same spline smoothing used by create_background_template in ixpeobssim.bkg.instr, and write it as an ascii energy-spectrum table (background_template.txt) usable as input to an xTemplateInstrumentalBkg object. The rationale is that the IXPE instrumental background is stable over time and shows no significant polarization, so a single observation-derived template is a valid representation.

## Related tools

- **ixpeobssim** (Provides the FITS utilities for reading pha1 headers (BACKSCAL, LIVETIME) and the create_background_template / xTemplateInstrumentalBkg machinery that consumes the resulting ascii template) — https://github.com/lucabaldini/ixpeobssim
- **Python** (Language in which the header-reading, rate normalization, averaging, and spline smoothing steps are implemented)

## Examples

```
from ixpeobssim.bkg.instr import create_background_template; create_background_template('background_template.txt')
```

## Evaluation signals

- background_template.txt exists and is parseable as an ascii energy-spectrum table.
- All smoothed rate values in background_template.txt are non-negative.
- Rate values are consistent with the livetime- and area-normalized input rates (i.e., rescaling by LIVETIME and BACKSCAL round-trips to physical units).
- The template loads as input to an xTemplateInstrumentalBkg object without error.
- Each input pha1 file's header keyword values (BACKSCAL, LIVETIME) were read from the file itself, not assumed.

## Limitations

- The single-template assumption rests on the instrumental background being stable over time and showing no significant polarization, per the authors' knowledge ('To our knowledge...').
- Solar flaring background can be sub-threshold yet persistently strongly polarized, violating the template's unpolarized-background premise.
- ixpeobssim interoperability depends on HEASOFT; mismatched environments may affect FITS handling.

## Evidence

- [other] Load the input pha1 spectrum file(s) for the dark field with ixpeobssim FITS utilities and read the backscal keyword and livetime from the header.: [evidence span withheld — anchor status: not_found]
- [methods] for the purpose of rescaling (e.g.: by the livetime or backscal) and subtracting data (as for background subtraction): "for the purpose of rescaling (e.g.: by the livetime or backscal) and subtracting data (as for background subtraction)"
- [methods] To our knowledge, this background component is stable over time and shows no significant polarization: "To our knowledge, this background component is stable over time and shows no significant polarization"
- [readme] ixpeobssim is a simulation and analysis framework specifically developed for the Imaging X-ray Polarimetry Explorer (IXPE).: "ixpeobssim is a simulation and analysis framework specifically developed for the Imaging X-ray Polarimetry Explorer (IXPE)."
- [methods] the solar activity has been proven to be often sub-threshold and carries a small number of counts that sometimes are however persistently strongly polarized: "the solar activity has been proven to be often sub-threshold and carries a small number of counts that sometimes are however persistently strongly polarized"
