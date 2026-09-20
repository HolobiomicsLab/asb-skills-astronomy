---
name: planetary-system-analogue-identification
description: Use when you have observed planetary parameters (radii ~3.41, 1.55, 1.54
  R_earth; periods 13.6, 3.5, 6.4 days for HIP 29442) plus plaNETic posterior samples
  and need to infer formation pathways by scanning a Bern-model synthetic population
  for system analogues.
when_to_use_negative:
- Planets are outside plaNETic's validated mass range of 0.5–15 Earth masses, so internal-structure
  posteriors feeding the analogue matching cannot be trusted.
- The goal is internal-structure inference itself rather than formation-pathway interpretation
  — use the plaNETic grid accept-reject sampling skill directly.
- No synthetic population or formation model output is available, so matched analogues
  cannot provide iceline or disk-solid-mass diagnostics.
tools:
- name: Bern model
  role: Generates the synthetic planet-formation population from which analogues are
    identified and formation properties (iceline location, disk solid mass) are extracted
- name: plaNETic
  role: Produces the Bayesian internal-structure posteriors ('xxx_posterior.npy')
    that supply the observed planet mass/radius constraints used for analogue matching
  repo: github.com/joannegger/plaNETic
- name: Python 3.9
  role: Runtime for the run_grid executables and the analogue scanning/matching script
  repo: github.com/joannegger/plaNETic
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:7b92af7877b2f929e1f2c847ccf92965f182b1aaaaae65363b90c7f975a33f9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bern model
  - Python
  - Python 3.9
  - plaNETic
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# planetary-system-analogue-identification

## Summary

Identifies planetary system analogues in a Bern-model synthetic planet-formation population by matching observed planet masses, radii, and periods (from plaNETic/CHEOPS analysis of HIP 29442, TOI-469), then extracting formation diagnostics such as formation location relative to the water iceline and initial disk solid mass.

## When to use

Use when you have observed planetary parameters (radii ~3.41, 1.55, 1.54 R_earth; periods 13.6, 3.5, 6.4 days for HIP 29442) plus plaNETic posterior samples and need to infer formation pathways by scanning a Bern-model synthetic population for system analogues.

## When NOT to use

- Planets are outside plaNETic's validated mass range of 0.5–15 Earth masses, so internal-structure posteriors feeding the analogue matching cannot be trusted.
- The goal is internal-structure inference itself rather than formation-pathway interpretation — use the plaNETic grid accept-reject sampling skill directly.
- No synthetic population or formation model output is available, so matched analogues cannot provide iceline or disk-solid-mass diagnostics.

## Inputs

- Bern-model synthetic planet-formation and evolution population
- 'stellar_planetary_parameters.csv' with observed host-star and planetary parameters of HIP 29442 (TOI-469)
- plaNETic posterior samples 'xxx_posterior.npy' for planets b, c, and d from the 'posteriors' subfolder

## Outputs

- 'hip29442_bern_analogues.csv' with per-planet and per-system formation diagnostics for matched analogues
- Labelled dominant formation scenario (planets formed on opposing sides of the water iceline from a disk with intermediate solid mass)

## How to apply

Load the Bern-model synthetic planet formation and evolution population, then load the observed host-star and planetary parameters of HIP 29442 from 'stellar_planetary_parameters.csv' together with plaNETic posterior samples ('xxx_posterior.npy') for planets b, c, and d generated via the 'run_grid' workflow. Scan the population for planetary system analogues by matching planet masses, radii, and orbital periods against the observed constraints (radii 3.410 ± 0.046, 1.551 ± 0.045, 1.538 ± 0.049 R_earth; periods 13.6, 3.5, 6.4 days). For each matched analogue, extract Bern-model formation properties, in particular each planet's formation location relative to the water iceline and the disk's initial solid mass. Label the dominant formation scenario, testing whether the planets formed on opposing sides of the water iceline from a disk with intermediate solid mass, and write per-planet and per-system formation diagnostics to 'hip29442_bern_analogues.csv'.

## Related tools

- **Bern model** (Generates the synthetic planet-formation population from which analogues are identified and formation properties (iceline location, disk solid mass) are extracted)
- **plaNETic** (Produces the Bayesian internal-structure posteriors ('xxx_posterior.npy') that supply the observed planet mass/radius constraints used for analogue matching) — github.com/joannegger/plaNETic
- **Python 3.9** (Runtime for the run_grid executables and the analogue scanning/matching script) — github.com/joannegger/plaNETic

## Examples

```
python run_grid/TOI-469_Egger+/run_grid_TOI-469.py  # generates xxx_posterior.npy for planets b, c, d for analogue matching
```

## Evaluation signals

- The matched analogue ensemble reproduces the paper's finding that the HIP 29442 planets likely formed on opposing sides of the water iceline from a protoplanetary disk with an intermediate solid mass; otherwise flag the scenario as not supported.
- Matched analogues are consistent with the observed radii 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R_earth and periods 13.6, 3.5, and 6.4 days for planets b, c, and d.
- Output file 'hip29442_bern_analogues.csv' exists and contains per-planet and per-system formation diagnostics for every matched analogue.
- Each match is anchored to plaNETic posterior samples ('xxx_posterior.npy') rather than discovery-paper values, since planet d's discovery-paper radius deviates by more than 3σ.

## Limitations

- If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage and posterior files needed for matching cannot be generated without contacting the authors.
- plaNETic is only valid for small exoplanets with masses between 0.5 and 15 Mearth, limiting which planets can be matched to analogues.
- The observed parameters are compatible with both a pure H/He-envelope scenario and a water-rich sub-Neptune scenario for the second peak of the radius bimodality, so analogue-based formation labels are not unique.
- Caution is required when using TESS photometry for small planets with low per-transit SNR and large gaps, as it affected the radius of planet d by more than 3σ.

## Evidence

- [abstract] we identified planetary system analogues in a synthetic population generated with the Bern model for planet formation and evolution: "we identified planetary system analogues in a synthetic population generated with the Bern model for planet formation and evolution"
- [abstract] the planets likely formed on opposing sides of the water iceline from a protoplanetary disk with an intermediate solid mass: "the planets likely formed on opposing sides of the water iceline from a protoplanetary disk with an intermediate solid mass"
- [abstract] Planetary radii of 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, and d with periods of 13.6, 3.5, and 6.4 days: "planetary radii of 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, and d, which orbit HIP 29442 with periods of 13.6, 3.5, and 6.4 days"
- [readme] create a new subfolder in 'run_grid' with the same structure as 'TOI-469_Egger+' or 'TOI-238_Egger+': "create a new subfolder in 'run_grid' with the same structure as 'TOI-469_Egger+' or 'TOI-238_Egger+'"
- [intro] Bayesian internal structure modelling framework for small exoplanets with masses between 0.5 and 15 Mearth: "Bayesian internal structure modelling framework for small exoplanets with masses between 0.5 and 15 Mearth"
