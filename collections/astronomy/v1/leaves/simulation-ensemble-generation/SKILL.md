---
name: simulation-ensemble-generation
description: Use when you need to gauge statistical properties of IXPE polarization
  estimates—e.g., posterior distributions, pull distributions, or spectro-polarimetric
  fit biases—by Monte Carlo sampling many realizations of the same observation rather
  than analyzing a single simulated event list.
when_to_use_negative:
- You only need a single realistic simulated observation or demonstration event list—one
  ixpeobssim simulation suffices; the ensemble machinery is for statistical-property
  studies.
- Disk space is severely constrained and you need the raw event lists retained—the
  default cleanup deletes intermediate event files after binning, so the ensemble
  workflow will not preserve them.
- The analysis targets a non-IXPE mission—the framework is IXPE-specific by default
  and adaptation to other polarization-capable missions is possible but non-trivial.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ixpeobssim
  role: Framework providing pipeline.bootstrap_pipeline, pipeline.generate_ensamble,
    standard_ensamble_processing, and xpbin for ensemble simulation and binning
  repo: https://github.com/lucabaldini/ixpeobssim
- name: XPBIN
  role: Binning application invoked per event list to produce PHA1/PHA1Q/PHA1U Stokes
    spectra and PCUBE modulation cubes
  repo: https://github.com/lucabaldini/ixpeobssim
- name: XSPEC
  role: Downstream tool used to fit the I, Q and U Stokes spectra from each realization
    with a 'powerlaw * polconst' spectro-polarimetric model and store the fit results
- name: IXPE (Imaging X-ray Polarimetry Explorer)
  role: Mission whose instrument response functions and detector units (DU) define
    the simulated observations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2022.101194
    title: 'ixpeobssim: A simulation and analysis framework for the imaging X-ray
      polarimetry explorer'
  source_content_hash: sha256:45da9a5d2dbdaad2ddf324f950c0121b07db50bdec428e19303b406093e14d15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lucabaldini/ixpeobssim
  tools:
  - IXPE (Imaging X-ray Polarimetry Explorer)
  - Python
  - XPBIN
  - XSPEC
  - ixpeobssim
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# simulation-ensemble-generation

## Summary

Generate large Monte Carlo ensembles of simulated IXPE observations with ixpeobssim (e.g., 1000 realizations of a 50 ks observation) to gauge the statistical properties of polarization estimators, yielding per-realization binned products (Stokes PHA1/PHA1Q/PHA1U spectra and modulation cubes) for downstream pull-distribution and XSPEC fitting analysis.

## When to use

Use when you need to gauge statistical properties of IXPE polarization estimates—e.g., posterior distributions, pull distributions, or spectro-polarimetric fit biases—by Monte Carlo sampling many realizations of the same observation rather than analyzing a single simulated event list.

## When NOT to use

- You only need a single realistic simulated observation or demonstration event list—one ixpeobssim simulation suffices; the ensemble machinery is for statistical-property studies.
- Disk space is severely constrained and you need the raw event lists retained—the default cleanup deletes intermediate event files after binning, so the ensemble workflow will not preserve them.
- The analysis targets a non-IXPE mission—the framework is IXPE-specific by default and adaptation to other polarization-capable missions is possible but non-trivial.

## Inputs

- ixpeobssim source model / ROI model configuration for the target source (e.g., the toy_pollin example)
- Instrument response functions (IXPE CALDB)
- Ensemble parameters: number of realizations (e.g., size=1000) and observation duration (e.g., 50000 s)

## Outputs

- Per-realization FITS event lists for the three IXPE detector units (deleted after default cleanup)
- Per-realization binned Stokes spectra in PHA1, PHA1Q, PHA1U flavors from xpbin
- Per-realization modulation-cube (PCUBE) polarization products
- Collected ensemble of polarization estimates for pull-distribution or XSPEC fit analysis

## How to apply

Bootstrap the source-model pipeline for the chosen configuration with ixpeobssim.core.pipeline.bootstrap_pipeline('toy_pollin'), which sets up the ROI model, instrument response, and working directory. Generate the ensemble with pipeline.generate_ensamble(size=1000., duration=50000.), producing a FITS event list for each of the three detector units per realization; each event list contains a strict superset of the information in the publicly released HEASARC IXPE data products. Let the default standard_ensamble_processing run xpbin on each event list to produce the three Stokes-spectrum flavors (PHA1, PHA1Q, PHA1U) and modulation-cube (PCUBE) polarization products per realization. Accept the default cleanup, which deletes the large intermediate event files after processing so only binned products remain on disk; re-simulate rather than re-binning if the raw event lists are needed again. Collect the per-realization binned polarization estimates and feed them into post-processing such as XSPEC spectro-polarimetric fitting (e.g., a 'powerlaw * polconst' model on the I, Q, and U Stokes spectra) to build pull or posterior parameter distributions.

## Related tools

- **ixpeobssim** (Framework providing pipeline.bootstrap_pipeline, pipeline.generate_ensamble, standard_ensamble_processing, and xpbin for ensemble simulation and binning) — https://github.com/lucabaldini/ixpeobssim
- **XPBIN** (Binning application invoked per event list to produce PHA1/PHA1Q/PHA1U Stokes spectra and PCUBE modulation cubes) — https://github.com/lucabaldini/ixpeobssim
- **XSPEC** (Downstream tool used to fit the I, Q and U Stokes spectra from each realization with a 'powerlaw * polconst' spectro-polarimetric model and store the fit results)
- **IXPE (Imaging X-ray Polarimetry Explorer)** (Mission whose instrument response functions and detector units (DU) define the simulated observations)

## Examples

```
from ixpeobssim.core import pipeline; pipeline.bootstrap_pipeline('toy_pollin'); pipeline.generate_ensamble(size=1000., duration=50000.)
```

## Evaluation signals

- Ensemble completeness: exactly 1000 realizations each yield the Stokes-spectrum (PHA1/PHA1Q/PHA1U) and modulation-cube binned products.
- Each simulated event list is FITS format and contains a strict superset of the information included in the publicly released IXPE data products.
- Products are generated for all three detector units of each realization.
- After default cleanup, only binned products remain on disk—no large intermediate event files should be present.
- Posterior/pull parameter distributions built from the ensemble are centered and scaled consistently with the injected ground-truth source model (checked via XSPEC fit results).

## Limitations

- The default ensemble cleanup deletes the intermediate event files after xpbin processing, so raw event lists are not retained unless cleanup is disabled.
- Ensemble runs are computationally heavy: 1000 realizations of a 50 ks observation multiply storage and runtime relative to a single simulation.
- The framework is largely IXPE-specific; while the modular design is potentially adaptable to other polarization-capable missions, that adaptation is not automatic.

## Evidence

- [methods] Ensemble generation rationale: "generate a large number of different realizations of the same observation to gauge the statistical properties"
- [readme] Event list fidelity: "produce realistic simulated observations, in the form of event lists in FITS format, containing a strict superset of the information included in the publicly released IXPE data products"
- [methods] Stokes-spectrum binning flavors: "binned files in the three flavors are created by invoking |xpbin| with the ``PHA1``, ``PHA1Q`` and ``PHA1U`` values"
- [methods] Downstream XSPEC fitting of ensemble products: "fit the Stokes spectra with XSPEC with the proper spectro-polarimetric model and store the fit results"
- [abstract] Filtering stages before final event list: "the partial event lists are then merged and sorted in time. All the additional instrumental corrections are applied in dedicated filtering stages before the final, consolidated event list is written"
