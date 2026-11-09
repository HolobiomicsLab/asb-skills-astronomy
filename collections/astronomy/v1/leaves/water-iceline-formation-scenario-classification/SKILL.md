---
name: water-iceline-formation-scenario-classification
description: Use when internal-structure posteriors exist (xxx_posterior.npy) for
  a multi-planet system and the goal is to decide whether planets formed inside vs.
  outside the water iceline from a low/intermediate/high-mass solid disk, via synthetic-population
  analogues.
when_to_use_negative:
- Planet masses fall outside plaNETic's validated range of 0.5–15 Mearth, so internal-structure
  posteriors cannot be reliably computed before the analogue search
- Only photometric radii without mass constraints (e.g. no ESPRESSO/HIRES RVs) are
  available, so analogue matching on planet masses cannot be performed
- No synthetic population from a planet-formation model (e.g. Bern model) is available,
  so no formation-history diagnostics can be extracted for matched analogues
edam_operation: http://edamontology.org/operation_2426
tools:
- name: plaNETic
  role: Neural-network Bayesian internal structure modelling framework; computes posterior
    samples used to constrain planet masses/radii and water-abundance priors (formation
    inside vs. outside of iceline) fed into the analogue search
  repo: github.com/joannegger/plaNETic
- name: Bern model
  role: Generates the synthetic planet-formation and evolution population in which
    system analogues are identified and from which iceline-side and disk-solid-mass
    formation properties are extracted
- name: CHEOPS
  role: Provides 9.6 days of photometry of HIP 29442 modelled jointly with TESS to
    derive the planetary radii used for analogue matching
- name: TESS
  role: Two sectors of photometry jointly modelled with CHEOPS to derive planetary
    radii
- name: Python 3.9
  role: Runtime environment for plaNETic (virtual environment, pip install, running
    run_grid executables)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:7b3ef994fdde8b7bb677598eb1bf40dcab75e8f6b74d24771f9815466d517718
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bern model
  - CHEOPS
  - Python
  - Python 3.9
  - TESS
  - plaNETic
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# water-iceline-formation-scenario-classification

## Summary

Classifies formation pathways of an observed multi-planet system by scanning a Bern-model synthetic planet population for analogues matched to plaNETic-derived radii, masses, and periods, and extracting each analogue's formation location relative to the water iceline and the disk's initial solid mass.

## When to use

Use when internal-structure posteriors exist (xxx_posterior.npy) for a multi-planet system and the goal is to decide whether planets formed inside vs. outside the water iceline from a low/intermediate/high-mass solid disk, via synthetic-population analogues.

## When NOT to use

- Planet masses fall outside plaNETic's validated range of 0.5–15 Mearth, so internal-structure posteriors cannot be reliably computed before the analogue search
- Only photometric radii without mass constraints (e.g. no ESPRESSO/HIRES RVs) are available, so analogue matching on planet masses cannot be performed
- No synthetic population from a planet-formation model (e.g. Bern model) is available, so no formation-history diagnostics can be extracted for matched analogues

## Inputs

- stellar_planetary_parameters.csv (observed host-star and planetary parameters for the system)
- xxx_posterior.npy internal-structure posterior samples from plaNETic for planets b, c, and d
- Bern-model synthetic planet-formation and evolution population
- Published ESPRESSO and HIRES radial velocities (mass constraints)

## Outputs

- hip29442_bern_analogues.csv — analogue table with per-planet and per-system formation diagnostics (iceline side, disk solid mass)
- Labelled dominant formation scenario (e.g. opposing sides of the water iceline, intermediate disk solid mass)

## How to apply

First derive planetary radii by jointly modelling CHEOPS photometry (9.6 days of observations for HIP 29442) with two TESS sectors and combining them with published ESPRESSO and HIRES RVs. Run plaNETic (neural networks trained on BICEPS interior models, full grid accept-reject sampling, applicable to 0.5–15 Mearth planets) on a run_grid subfolder containing stellar_planetary_parameters.csv to obtain xxx_posterior.npy samples for each planet. Then load the Bern-model synthetic population and scan it for planetary system analogues by matching planet masses, radii, and orbital periods against the observed constraints (for HIP 29442: radii ~3.41, 1.55, 1.54 Rearth; periods 13.6, 3.5, 6.4 days). For each matched analogue, extract the formation location relative to the water iceline and the protoplanetary disk's initial solid mass, then label the dominant scenario. For HIP 29442 the expected label is planets formed on opposing sides of the water iceline from a disk with intermediate solid mass; write per-planet and per-system diagnostics to hip29442_bern_analogues.csv.

## Related tools

- **plaNETic** (Neural-network Bayesian internal structure modelling framework; computes posterior samples used to constrain planet masses/radii and water-abundance priors (formation inside vs. outside of iceline) fed into the analogue search) — github.com/joannegger/plaNETic
- **Bern model** (Generates the synthetic planet-formation and evolution population in which system analogues are identified and from which iceline-side and disk-solid-mass formation properties are extracted)
- **CHEOPS** (Provides 9.6 days of photometry of HIP 29442 modelled jointly with TESS to derive the planetary radii used for analogue matching)
- **TESS** (Two sectors of photometry jointly modelled with CHEOPS to derive planetary radii)
- **Python 3.9** (Runtime environment for plaNETic (virtual environment, pip install, running run_grid executables))

## Examples

```
cd run_grid/TOI-469_Egger+ && python run_grid_TOI-469.py  # generates b/c/d posterior .npy samples for iceline-side analogue matching
```

## Evaluation signals

- The derived radii match the published values 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 Rearth for planets b, c, and d with periods 13.6, 3.5, and 6.4 days
- xxx_posterior.npy files are generated in the 'posteriors' subfolder for all three planets and all prior options (A1–B3 style water/Si-Mg-Fe prior choices)
- hip29442_bern_analogues.csv contains per-planet and per-system formation diagnostics including iceline side and disk initial solid mass for every matched analogue
- The matched analogue ensemble reproduces the paper's finding of formation on opposing sides of the water iceline from a disk with intermediate solid mass; otherwise the scenario is flagged as not supported
- Inference output shapes are non-empty posterior arrays (e.g. (1, N, 28) as in the README's TOI-238 b example)

## Limitations

- Planet d's radius deviates by more than 3σ from the discovery paper's median value, so caution is required when using TESS photometry for small planets with low per-transit SNR and large data gaps
- If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage and the authors must be contacted
- plaNETic needs GPU access to run efficiently; CPU-only runs drastically increase computation time (10–15 min per planet per prior option on an Apple M4 Pro)
- Observed parameters are compatible with both a pure H/He-envelope sub-Neptune scenario and a water-rich sub-Neptune scenario for the second peak of the radius bimodality, so the analogue classification may be degenerate

## Evidence

- [abstract] we identified planetary system analogues in a synthetic population generated with the Bern model for planet formation and evolution: "we identified planetary system analogues in a synthetic population generated with the Bern model for planet formation and evolution"
- [abstract] the planets likely formed on opposing sides of the water iceline from a protoplanetary disk with an intermediate solid mass: "the planets likely formed on opposing sides of the water iceline from a protoplanetary disk with an intermediate solid mass"
- [intro] it allows for different choices in priors concerning the expected abundance of water (formation inside vs. outside of iceline): "it allows for different choices in priors concerning the expected abundance of water (formation inside vs. outside of iceline)"
- [readme] Then simply adapt and run the executable, which will generate 'xxx_posterior.npy' files in a subfolder 'posteriors'.: "Then simply adapt and run the executable, which will generate 'xxx_posterior.npy' files in a subfolder 'posteriors'."
- [intro] Bayesian internal structure modelling framework for small exoplanets with masses between 0.5 and 15 Mearth: "Bayesian internal structure modelling framework for small exoplanets with masses between 0.5 and 15 Mearth"
