---
name: protoplanetary-disk-solid-mass-diagnosis
description: Use when an observed multi-planet system has plaNETic internal-structure
  posteriors (e.g. xxx_posterior.npy from run_grid) and you need to infer formation
  history — iceline position and disk solid mass — from a Bern-model synthetic population.
when_to_use_negative:
- Planet masses fall outside the 0.5–15 Mearth range that plaNETic's neural networks
  (trained on BICEPS) were built for, so reliable internal-structure posteriors —
  the input constraints for the analogue match — cannot be produced.
- The question is purely observational (e.g. radius or period refinement from photometry)
  with no need for formation history; the analogue scan adds disk-formation interpretation
  only.
- No synthetic population with recorded formation properties (iceline formation location,
  disk solid mass) is available to match against.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: plaNETic
  role: Neural-network Bayesian internal structure modelling framework producing the
    posterior samples of radii/structure used as observational constraints when scanning
    the population for analogues
  repo: https://github.com/joannegger/plaNETic
- name: Bern model
  role: Generates the synthetic planet-formation and evolution population whose analogues
    are matched and whose iceline-formation and disk-solid-mass diagnostics are extracted
- name: Python
  role: Implementation language for the analogue scan (Python 3.9 environment recommended
    in README)
  repo: https://github.com/joannegger/plaNETic
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:389fe194b617a7c501250f53eac1e02092b0b0356d92f1fdb56075cfd16365ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannegger/plaNETic
  tools:
  - Bern model
  - Python
  - plaNETic
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# protoplanetary-disk-solid-mass-diagnosis

## Summary

Diagnoses the formation environment (position relative to the water iceline) and the initial protoplanetary disk solid mass of an observed planetary system by scanning a Bern-model synthetic planet-formation population for system analogues matched against observed radii, masses, and periods.

## When to use

Use when an observed multi-planet system has plaNETic internal-structure posteriors (e.g. xxx_posterior.npy from run_grid) and you need to infer formation history — iceline position and disk solid mass — from a Bern-model synthetic population.

## When NOT to use

- Planet masses fall outside the 0.5–15 Mearth range that plaNETic's neural networks (trained on BICEPS) were built for, so reliable internal-structure posteriors — the input constraints for the analogue match — cannot be produced.
- The question is purely observational (e.g. radius or period refinement from photometry) with no need for formation history; the analogue scan adds disk-formation interpretation only.
- No synthetic population with recorded formation properties (iceline formation location, disk solid mass) is available to match against.

## Inputs

- stellar_planetary_parameters.csv with observed host-star and planetary parameters
- plaNETic posterior samples xxx_posterior.npy for planets b, c, and d (posteriors subfolder)
- Bern-model synthetic planet-formation population (planet formation and evolution)

## Outputs

- hip29442_bern_analogues.csv — analogue table with per-planet and per-system formation diagnostics
- Dominant formation scenario label (e.g. 'opposing sides of the water iceline, intermediate disk solid mass') or a 'not supported' flag

## How to apply

First derive observational constraints by running plaNETic with stellar_planetary_parameters.csv to obtain xxx_posterior.npy samples for each planet (here HIP 29442 b, c, d with radii 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ and periods 13.6, 3.5, and 6.4 days). Then load the Bern-model synthetic planet-formation population and scan it for system analogues by matching planet masses, radii, and orbital periods against the plaNETic-constrained observations. For every matched analogue, extract the Bern-model formation diagnostics — each planet's formation location relative to the water iceline and the disk's initial solid mass — and label the dominant scenario across the ensemble. Accept the diagnosis only if the matched analogues support the tested scenario (for HIP 29442: opposing sides of the water iceline from an intermediate-solid-mass disk); otherwise flag the scenario as not supported by the population. Write per-planet and per-system formation diagnostics to an output table such as hip29442_bern_analogues.csv.

## Related tools

- **plaNETic** (Neural-network Bayesian internal structure modelling framework producing the posterior samples of radii/structure used as observational constraints when scanning the population for analogues) — https://github.com/joannegger/plaNETic
- **Bern model** (Generates the synthetic planet-formation and evolution population whose analogues are matched and whose iceline-formation and disk-solid-mass diagnostics are extracted)
- **Python** (Implementation language for the analogue scan (Python 3.9 environment recommended in README)) — https://github.com/joannegger/plaNETic

## Examples

```
cd run_grid/TOI-238_Egger+ && python run_grid_TOI-238.py
```

## Evaluation signals

- The matched analogue ensemble reproduces the paper's finding that the planets likely formed on opposing sides of the water iceline from a protoplanetary disk with an intermediate solid mass; otherwise the scenario is flagged as not supported.
- Each matched analogue satisfies simultaneous consistency with observed masses, radii, and orbital periods for planets b, c, and d (periods 13.6, 3.5, 6.4 days).
- hip29442_bern_analogues.csv contains both per-planet and per-system formation diagnostics for every analogue.
- Upstream xxx_posterior.npy files exist in the 'posteriors' subfolder with plausible sample shapes, e.g. posteriors of shape (1, 771909, 28) per prior option as in the README's TOI-238 example.
- Reported radii agree with the CHEOPS+TESS joint-fit values (3.410 ± 0.046, 1.551 ± 0.045, 1.538 ± 0.049 R⊕) rather than the discovery-paper TESS-only values for planet d.

## Limitations

- Caution is required when using TESS photometry for small planets with low per-transit SNR and large gaps: planet d's radius deviated by more than 3σ from the discovery paper's median value, which propagates into the analogue-matching constraints.
- plaNETic needs GPU access to run efficiently; CPU-only runs drastically increase computation time and are not recommended (inference is ~10–15 min per planet per prior option even on an Apple M4 Pro GPU).
- If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage and the author must be contacted.
- Internal-structure posteriors remain compatible with both pure-H/He-envelope and water-rich sub-Neptune scenarios for the second peak of the radius bimodality, so water-prior choice (formation inside vs. outside iceline) affects interpretation.

## Evidence

- [abstract] the planets likely formed on opposing sides of the water iceline from a protoplanetary disk with an intermediate solid mass: "the planets likely formed on opposing sides of the water iceline from a protoplanetary disk with an intermediate solid mass"
- [abstract] analogue search in Bern population: "we identified planetary system analogues in a synthetic population generated with the Bern model for planet formation and evolution"
- [methods] plaNETic posterior generation workflow: "Then simply adapt and run the executable, which will generate 'xxx_posterior.npy' files in a subfolder 'posteriors'."
- [readme] DNN loading failure mode: "If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage."
- [abstract] radius deviation caveat: "For planet d this value deviates by more than 3σ from the median value reported in the discovery paper, leading us to conclude that caution is required when using TESS photometry"
