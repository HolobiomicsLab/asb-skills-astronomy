---
name: observation-configuration-from-pha-file
description: Use when a real X-ray spectrum (PHA file in the OGIP standard, e.g. the
  XMM-Newton NGC 7793 ULX-4 PN observation) must be ingested into jaxspec with an
  explicit energy band before fitting or mock-data generation.
when_to_use_negative:
- You are generating synthetic observations from an instrument definition rather than
  real data — use ObsConfiguration.mock_from_instrument(instrument, exposure=1e5)
  instead
- The spectrum is not in the OGIP standard (e.g. raw event files or non-X-ray spectroscopy
  formats) — jaxspec loads only OGIP-standard X-ray spectra
- You only need background handling (SubtractedBackground, BackgroundWithError, SpectralModelBackground)
  on an already-loaded ObsConfiguration — this skill covers ingestion, not background
  treatment
edam_operation: http://edamontology.org/operation_2409
tools:
- name: jaxspec
  role: Provides ObsConfiguration.from_pha_file to ingest OGIP-standard PHA files
    with band limits and no HEASoft dependency
  repo: https://github.com/renecotyfanboy/jaxspec
- name: Python
  role: Runtime environment (Python 3.11/3.12 supported; recommended fresh Python
    3.12 conda environment) in which jaxspec is installed via pip and the ObsConfiguration
    API is called
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:97d589dd8c757fcb84263e42ea90651020bfd6bd8519ec9845c499302ddfda30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/renecotyfanboy/jaxspec
  tools:
  - JAX
  - Python
  - jaxspec
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# observation-configuration-from-pha-file

## Summary

Load an X-ray observation stored as an OGIP-standard PHA file into a jaxspec ObsConfiguration, optionally restricted to an energy band (e.g. low_energy=0.3, high_energy=12 keV), so the instrument response, channels and counts are ready for model building, Bayesian fitting or fakeit-style mock generation.

## When to use

Use when a real X-ray spectrum (PHA file in the OGIP standard, e.g. the XMM-Newton NGC 7793 ULX-4 PN observation) must be ingested into jaxspec with an explicit energy band before fitting or mock-data generation.

## When NOT to use

- You are generating synthetic observations from an instrument definition rather than real data — use ObsConfiguration.mock_from_instrument(instrument, exposure=1e5) instead
- The spectrum is not in the OGIP standard (e.g. raw event files or non-X-ray spectroscopy formats) — jaxspec loads only OGIP-standard X-ray spectra
- You only need background handling (SubtractedBackground, BackgroundWithError, SpectralModelBackground) on an already-loaded ObsConfiguration — this skill covers ingestion, not background treatment

## Inputs

- PHA file following the OGIP standard (e.g. XMM-Newton NGC 7793 ULX-4 PN spectrum)
- Energy band limits in keV (low_energy, high_energy; e.g. 0.3 and 12)

## Outputs

- jaxspec ObsConfiguration object exposing the observation's instrument channels, counts and restricted energy band
- Downstream-ready configuration accepted by MCMCFitter/variational fitters and by fakeit_for_multiple_parameters

## How to apply

Call ObsConfiguration.from_pha_file on the PHA file, passing the band limits low_energy and high_energy in keV (the paper's example uses low_energy=0.3, high_energy=12 for the NGC 7793 ULX-4 PHA file); the resulting ObsConfiguration carries the instrument channel grid, counts and response information that downstream steps consume. The PHA file must follow the OGIP standard, which jaxspec reads natively without any HEASoft dependency, so no external preprocessing is needed. The returned obsconf is the first argument to fitters and to jaxspec.data.util.fakeit_for_multiple_parameters, so verify the band cut before parameter inference: any channels outside 0.3–12 keV are excluded from the likelihood. When the goal is mock spectra rather than a real observation, use ObsConfiguration.mock_from_instrument(instrument, exposure=1e5) instead of from_pha_file.

## Related tools

- **jaxspec** (Provides ObsConfiguration.from_pha_file to ingest OGIP-standard PHA files with band limits and no HEASoft dependency) — https://github.com/renecotyfanboy/jaxspec
- **Python** (Runtime environment (Python 3.11/3.12 supported; recommended fresh Python 3.12 conda environment) in which jaxspec is installed via pip and the ObsConfiguration API is called)

## Examples

```
from jaxspec.data import ObsConfiguration; obs = ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12)
```

## Evaluation signals

- The ObsConfiguration's channel grid reflects only the requested band (e.g. 0.3–12 keV for the NGC 7793 ULX-4 PHA file), confirming the low_energy/high_energy cuts were applied
- The input PHA file opens under jaxspec without HEASoft, confirming OGIP-standard compliance of the file
- The resulting obsconf can be passed directly as the first argument to fakeit_for_multiple_parameters and returns Poisson-realized mock counts per instrument channel without shape errors
- The loaded counts match the PHA file's observed counts within the selected band when compared against a FITS-level inspection of the same file

## Limitations

- jaxspec is still in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities, so the ObsConfiguration API may change between versions
- Only OGIP-standard X-ray spectra can be loaded; other formats are out of scope
- For observations with background, note the paper's caveat that the SubtractedBackground model accounts for observed counts without propagating any dispersion, which is inadequate for a comprehensive error budget — the ObsConfiguration step itself does not solve background uncertainty handling

## Evidence

- [methods] ObsConfiguration.from_pha_file usage with band limits: "obs = ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12)"
- [intro] OGIP-standard loading capability of jaxspec: "It can currently load an X-ray spectrum (in the OGIP standard), define a spectral model from the implemented components, and calculate the best parameters using state-of-the-art Bayesian approaches."
- [readme] jaxspec needs no HEASoft dependency for loading: [evidence span withheld — anchor status: external_doc]
- [methods] Mock observation alternative constructor: "obsconf = ObsConfiguration.mock_from_instrument(instrument, exposure=1e5,)"
