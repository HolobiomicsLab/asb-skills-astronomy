---
name: ixpeobssim-pipeline-configuration
description: Use when you need to generate simulated IXPE observations or an ensemble
  of realizations of a fixed-duration exposure (e.g. 1000 realizations of 50 ks) with
  FITS event lists and binned polarization products as outputs.
when_to_use_negative:
- You are analyzing real, publicly released IXPE HEASARC level-1/high-level data rather
  than producing simulated observations
- You need a non-IXPE polarimetry mission simulation without first adapting the modular
  framework to that mission's response functions
- You need raw event lists retained on disk, since the default ensemble cleanup deletes
  intermediate event files after xpbin processing
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: ixpeobssim
  role: 'Core simulation and analysis framework: bootstrap_pipeline and generate_ensamble
    produce the FITS event lists; xpbin produces the binned Stokes-spectra and modulation-cube
    products'
  repo: https://github.com/lucabaldini/ixpeobssim
- name: XSPEC
  role: Downstream fitter for the I, Q and U Stokes spectra (e.g. with a powerlaw
    * polconst spectro-polarimetric model) after ensemble generation
- name: HEASOFT
  role: Standard X-ray analysis environment with which the ixpeobssim output products
    are designed to interoperate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:0aeae2a8147273cf7f2ba38546732bd75b1e713ea3b55da98f7464abc900342d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - HEASOFT
  - Python
  - XSPEC
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# ixpeobssim-pipeline-configuration

## Summary

Configure and run the ixpeobssim simulation/analysis pipeline for IXPE: bootstrap a source-model pipeline (e.g. toy_pollin), Monte Carlo an ensemble of realizations of a fixed exposure, bin event lists into Stokes-spectra and modulation-cube products, and clean up intermediates. This yields the per-realization binned polarization products from which statistical (pull/posterior) distributions can be gauged.

## When to use

Use when you need to generate simulated IXPE observations or an ensemble of realizations of a fixed-duration exposure (e.g. 1000 realizations of 50 ks) with FITS event lists and binned polarization products as outputs.

## When NOT to use

- You are analyzing real, publicly released IXPE HEASARC level-1/high-level data rather than producing simulated observations
- You need a non-IXPE polarimetry mission simulation without first adapting the modular framework to that mission's response functions
- You need raw event lists retained on disk, since the default ensemble cleanup deletes intermediate event files after xpbin processing

## Inputs

- Source/ROI model specification (e.g. the toy_pollin source model) and IXPE instrument response functions (CALDB)
- Ensemble parameters: number of realizations (size, e.g. 1000) and observation duration (e.g. 50000 s)
- Environment variables such as IXPEOBSSIM_REPOSITORY and CALDBVER

## Outputs

- FITS event lists for the three detector units per realization (a strict superset of the publicly released IXPE data products)
- Binned Stokes spectra in PHA1, PHA1Q and PHA1U flavors per realization
- Modulation-cube polarization products (e.g. PCUBE) per realization
- Only binned products on disk after the default cleanup deletes intermediate event files

## How to apply

Bootstrap the source-model pipeline with ixpeobssim.core.pipeline.bootstrap_pipeline('<source_name>') to set up the ROI model, instrument response functions, and working directory (e.g. 'toy_pollin'). Call pipeline.generate_ensamble(size=1000., duration=50000.) to Monte Carlo many realizations of the same observation, each producing a FITS event list for the three detector units; the simulation flow convolves the photon spectrum with the instrument response, extracts the photo-electron emission angle from the proper azimuthal distribution, merges and sorts partial event lists in time, and applies filtering stages (vignetting by event rejection vs off-axis angle, discarding events outside the chip fiducial area and events during earlier-event readout) before writing the final event list. Let the default standard_ensamble_processing run xpbin on each event list to create binned products in the PHA1, PHA1Q and PHA1U flavors (Stokes spectra) plus modulation-cube products (PCUBE). Allow the default cleanup to delete the large intermediate event files so only binned products remain. Collect the per-realization binned polarization estimates for downstream steps such as XSPEC spectro-polarimetric fitting (e.g. powerlaw * polconst) and pull-distribution analysis.

## Related tools

- **ixpeobssim** (Core simulation and analysis framework: bootstrap_pipeline and generate_ensamble produce the FITS event lists; xpbin produces the binned Stokes-spectra and modulation-cube products) — https://github.com/lucabaldini/ixpeobssim
- **XSPEC** (Downstream fitter for the I, Q and U Stokes spectra (e.g. with a powerlaw * polconst spectro-polarimetric model) after ensemble generation)
- **HEASOFT** (Standard X-ray analysis environment with which the ixpeobssim output products are designed to interoperate)

## Examples

```
from ixpeobssim.core import pipeline; pipeline.bootstrap_pipeline('toy_pollin'); pipeline.generate_ensamble(size=1000., duration=50000.)
```

## Evaluation signals

- The ensemble is complete when all 1000 realizations each yield the Stokes-spectra (PHA1/PHA1Q/PHA1U) and modulation-cube binned products
- Each realization produces FITS event lists for the three detector units before xpbin processing
- After the default cleanup, only binned polarization products remain on disk (intermediate event files deleted)
- The per-realization binned estimates form a sample from which posterior/pull distributions of the model parameters can be gauged
- Simulated event lists contain a strict superset of the information included in the publicly released IXPE data products

## Limitations

- Much of the framework is specific to IXPE; adapting it to other polarization-capable missions relies on its modular nature and is 'potentially straightforward' rather than guaranteed
- Large intermediate event files are deleted by the default cleanup, so raw per-realization event lists are not preserved unless you disable cleanup
- No changelog/discussion content was available in the article, so operational caveats beyond the documented workflow are under-specified

## Evidence

- [abstract] it is designed to produce realistic simulated observations, in the form of event lists in FITS format, containing a strict superset of the information included in the publicly released IXPE data: "it is designed to produce realistic simulated observations, in the form of event lists in FITS format, containing a strict superset of the information included in the publicly released IXPE data"
- [methods] generate a large number of different realizations of the same observation to gauge the statistical properties: "generate a large number of different realizations of the same observation to gauge the statistical properties"
- [methods] binned files in the three flavors are created by invoking |xpbin| with the ``PHA1``, ``PHA1Q`` and ``PHA1U`` values: "binned files in the three flavors are created by invoking |xpbin| with the ``PHA1``, ``PHA1Q`` and ``PHA1U`` values"
- [abstract] All the additional instrumental corrections are applied in dedicated filtering stages before the final, consolidated event list is written: "All the additional instrumental corrections are applied in dedicated filtering stages before the final, consolidated event list is written"
- [readme] Given a source model and the response functions of the telescopes, it is designed to produce realistic simulated observations: "Given a source model and the response functions of the telescopes, it is designed to produce realistic simulated observations"
