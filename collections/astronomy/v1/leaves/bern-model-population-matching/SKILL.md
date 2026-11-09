---
name: bern-model-population-matching
description: Use when observed planetary parameters (radii, masses, periods) for a
  multi-planet system are available and the goal is to constrain formation history
  by finding analogues in a synthetic planet-formation population.
when_to_use_negative:
- Only radii are available without masses or posterior-based mass constraints, since
  analogue matching requires planet masses, radii, and periods jointly.
- The target planets fall outside the plaNETic validity range of 0.5–15 Mearth, so
  internal-structure posteriors cannot be reliably derived before population matching.
- No synthetic Bern-model population covering the relevant planet architectures is
  available for the system being studied.
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Bern model
  role: Generates the synthetic planet-formation population from which analogues are
    drawn and formation diagnostics (iceline-relative formation location, disk solid
    mass) are extracted.
- name: plaNETic
  role: Produces the Bayesian internal-structure posterior samples (xxx_posterior.npy)
    that supply the observed mass/radius constraints used to match population analogues.
  repo: https://github.com/joannegger/plaNETic
- name: Python
  role: Implementation language for the analogue-scanning workflow and CSV output.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:6272db218e41f58e39ac636bea02b28726ebba4b71f019d7e9646f8c42c43664
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannegger/plaNETic
  tools:
  - Bern model
  - Python
  - plaNETic
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bern-model-population-matching

## Summary

Identifies planetary-system analogues in a Bern-model synthetic planet-formation population by matching observed planet masses, radii, and orbital periods, then extracts formation diagnostics (formation location relative to the water iceline, initial disk solid mass) to infer formation pathways.

## When to use

Use when observed planetary parameters (radii, masses, periods) for a multi-planet system are available and the goal is to constrain formation history by finding analogues in a synthetic planet-formation population.

## When NOT to use

- Only radii are available without masses or posterior-based mass constraints, since analogue matching requires planet masses, radii, and periods jointly.
- The target planets fall outside the plaNETic validity range of 0.5–15 Mearth, so internal-structure posteriors cannot be reliably derived before population matching.
- No synthetic Bern-model population covering the relevant planet architectures is available for the system being studied.

## Inputs

- Bern-model synthetic planet-formation and evolution population
- stellar_planetary_parameters.csv with observed host-star and planetary parameters (e.g. HIP 29442 / TOI-469 planets b, c, d)
- plaNETic posterior samples ('xxx_posterior.npy') for each planet, providing mass/radius constraints

## Outputs

- Analogue table with per-planet and per-system formation diagnostics (e.g. hip29442_bern_analogues.csv)
- Dominant formation scenario label (e.g. planets formed on opposing sides of the water iceline from a disk with intermediate solid mass)

## How to apply

First run a Bayesian internal-structure framework (plaNETic) on the observed stellar/planetary parameters to obtain posterior samples ('xxx_posterior.npy') for each planet, as done for HIP 29442 planets b, c, d with radii ~3.41, 1.55, and 1.54 R_earth and periods 13.6, 3.5, and 6.4 days. Load the Bern-model synthetic population generated for planet formation and evolution, then scan it for system analogues by matching planet masses, radii, and orbital periods against the observed constraints (including the posterior distributions). For each matched analogue, extract formation properties from the Bern model — specifically each planet's formation location relative to the water iceline and the protoplanetary disk's initial solid mass — and label the dominant formation scenario across the matched ensemble. Write per-planet and per-system formation diagnostics to an output table (e.g. 'hip29442_bern_analogues.csv'). Accept the scenario if the analogue ensemble reproduces the paper's finding (opposing sides of the water iceline from an intermediate-solid-mass disk); otherwise flag it as not supported.

## Related tools

- **Bern model** (Generates the synthetic planet-formation population from which analogues are drawn and formation diagnostics (iceline-relative formation location, disk solid mass) are extracted.)
- **plaNETic** (Produces the Bayesian internal-structure posterior samples (xxx_posterior.npy) that supply the observed mass/radius constraints used to match population analogues.) — https://github.com/joannegger/plaNETic
- **Python** (Implementation language for the analogue-scanning workflow and CSV output.)

## Examples

```
cd run_grid/TOI-469_Egger+ && python run_grid_TOI-469.py  # generates posteriors for HIP 29442 b, c, d used as population-matching constraints
```

## Evaluation signals

- The matched analogue ensemble reproduces the reported finding that the planets likely formed on opposing sides of the water iceline from a protoplanetary disk with an intermediate solid mass; otherwise the scenario is flagged as not supported.
- Matched analogues are consistent with the observed radii (3.410 ± 0.046, 1.551 ± 0.045, 1.538 ± 0.049 R_earth) and periods (13.6, 3.5, 6.4 days) of planets b, c, and d.
- Output CSV contains per-planet and per-system formation diagnostics including iceline-relative formation location and initial disk solid mass for every analogue.
- Posterior .npy files for all three planets exist and were generated jointly by plaNETic before matching.

## Limitations

- Planet d's radius deviates by more than 3σ from the discovery paper's median value, so caution is required when using TESS photometry for small planets with low per-transit SNR and large gaps — radius uncertainties propagate into analogue matching.
- plaNETic requires GPU access to run efficiently; CPU-only runs drastically increase computation time.
- If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage and the authors must be contacted.
- Observed parameters may be compatible with more than one internal-structure scenario (pure H/He envelope vs. water-rich sub-Neptunes), so formation labels from population matching are not unique.

## Evidence

- [abstract] Analogues matched against observed HIP 29442 constraints: "we identified planetary system analogues in a synthetic population generated with the Bern model for planet formation and evolution"
- [abstract] Formation scenario label: "the planets likely formed on opposing sides of the water iceline from a protoplanetary disk with an intermediate solid mass"
- [methods] Posterior inputs from plaNETic: "Then simply adapt and run the executable, which will generate 'xxx_posterior.npy' files in a subfolder 'posteriors'."
- [intro] Mass range validity: "Bayesian internal structure modelling framework for small exoplanets with masses between 0.5 and 15 Mearth"
- [abstract] Radius caveat affecting matching constraints: "For planet d this value deviates by more than 3σ from the median value reported in the discovery paper, leading us to conclude that caution is required when using TESS photometry"
- [readme] Repository setup for run: "create a new subfolder in 'run_grid' with the same structure as 'TOI-469_Egger+' or 'TOI-238_Egger+'"
