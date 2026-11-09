---
name: ogip-standard-spectrum-loading
description: Use when the input is an X-ray observation delivered as a PHA file with
  ARF/RMF response files in OGIP format (e.g. from Hitomi or XMM-Newton) and you need
  to prepare it for spectral fitting in jaxspec.
when_to_use_negative:
- Data is not an X-ray spectrum in OGIP standard (e.g. already-binned model spectra
  or time series)
- You need a comprehensive error budget but plan to use SubtractedBackground, which
  does not propagate background dispersion
- You require HEASoft/XSPEC-specific processing chains rather than a pure-Python fitting
  path
edam_operation: http://edamontology.org/operation_0334
tools:
- name: jaxspec
  role: Pure-Python X-ray spectral fitting library that loads OGIP-standard spectra
    via ObsConfiguration
  repo: https://github.com/renecotyfanboy/jaxspec
- name: Python 3.12
  role: Recommended clean environment (conda or fresh Python) in which jaxspec is
    pip-installed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:be00a93d76d73d1c6788b7a1d8841381e9e1b1092f997f5582be83f19141ac6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/renecotyfanboy/jaxspec
  tools:
  - JAX
  - Python
  - Python 3.12
  - jaxspec
  edam_operation: http://edamontology.org/operation_0334
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# ogip-standard-spectrum-loading

## Summary

Load X-ray spectra stored in the OGIP standard into jaxspec by constructing an ObsConfiguration from the PHA file together with its ARF/RMF responses, band-limiting the fit range, optionally attaching background handling — the prerequisite step for any Bayesian spectral fit with jaxspec.

## When to use

Use when the input is an X-ray observation delivered as a PHA file with ARF/RMF response files in OGIP format (e.g. from Hitomi or XMM-Newton) and you need to prepare it for spectral fitting in jaxspec.

## When NOT to use

- Data is not an X-ray spectrum in OGIP standard (e.g. already-binned model spectra or time series)
- You need a comprehensive error budget but plan to use SubtractedBackground, which does not propagate background dispersion
- You require HEASoft/XSPEC-specific processing chains rather than a pure-Python fitting path

## Inputs

- PHA file in OGIP standard (e.g. Hitomi Perseus cluster observation)
- ARF/RMF instrument response files associated with the PHA file
- Optional background PHA file and low_energy/high_energy band limits (e.g. 0.3–12 keV)

## Outputs

- jaxspec ObsConfiguration object binding spectrum, response and band range
- Chosen background model object (SubtractedBackground, BackgroundWithError or SpectralModelBackground)

## How to apply

Start from a fresh Python 3.12 environment and `pip install jaxspec`; jaxspec is pure Python with no HEASoft dependency. Use `ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12)` to load the PHA spectrum and bind its ARF/RMF response, using the low_energy/high_energy arguments to band-limit the fit range before any inference. Choose a background treatment deliberately: `SubtractedBackground` (simple observed counts, no dispersion propagation), `BackgroundWithError`, or `SpectralModelBackground` — the article warns that SubtractedBackground gives an incomplete error budget. The resulting ObsConfiguration is then passed to a fitter (e.g. MCMCFitter.fit with num_chains/num_warmup/num_samples) or used for mock generation via `ObsConfiguration.mock_from_instrument`.

## Related tools

- **jaxspec** (Pure-Python X-ray spectral fitting library that loads OGIP-standard spectra via ObsConfiguration) — https://github.com/renecotyfanboy/jaxspec
- **Python 3.12** (Recommended clean environment (conda or fresh Python) in which jaxspec is pip-installed)

## Examples

```
from jaxspec.data import ObsConfiguration; obs = ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12)
```

## Evaluation signals

- ObsConfiguration is built without HEASoft dependencies and reports the requested band limits (e.g. low_energy=0.3, high_energy=12 keV)
- The ARF/RMF response is correctly associated with the PHA file (subsequent fit posterior plots, e.g. result.plot_ppc, show a plausible fit)
- Background model choice is recorded explicitly and its known dispersion limitation is acknowledged if SubtractedBackground is used
- Downstream fitting (MCMCFitter.fit) runs on the loaded configuration and reproduces published parameter values (e.g. Hitomi Perseus results)

## Limitations

- jaxspec is in early release: expect bugs, breaking API changes, undocumented features and missing functionalities
- SubtractedBackground does not propagate dispersion, so error budgets are incomplete when it is used
- Poisson likelihood (C-stat) should be preferred over chi-square at low counts to avoid biased parameter estimates

## Evidence

- [methods] obs = ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12): "obs = ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12)"
- [intro] It can currently load an X-ray spectrum (in the OGIP standard): "It can currently load an X-ray spectrum (in the OGIP standard), define a spectral model from the implemented components, and calculate the best parameters"
- [methods] The `SubtractedBackground` simply account for the observed counts without propagating any kind of dispersion: "The `SubtractedBackground` simply account for the observed counts without propagating any kind of dispersion, which is clearly bad when we want to get spectral parameters with a comprehensive error"
- [readme] jaxspec is written in pure Python, and has no dependancy to HEASoft: [evidence span withheld — anchor status: external_doc]
