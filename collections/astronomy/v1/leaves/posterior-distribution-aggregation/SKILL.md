---
name: posterior-distribution-aggregation
description: Use when grid/accept-reject sample posteriors exist per planet and prior
  option (e.g. plaNETic *_posterior.npy files) and you need weighted summary statistics
  (median, credible intervals) of derived quantities such as mass-loss rates.
when_to_use_negative:
- The observed radii you condition on are unreliable — the paper warns that TESS-only
  radii for small planets with low per-transit SNR and large gaps can be biased (planet
  d deviates >3σ from the discovery paper).
- Planet masses fall outside the 0.5–15 Earth-mass range that the plaNETic neural-network
  surrogate was trained for.
- Only raw posterior samples are needed without downstream derived quantities — simply
  load the .npy arrays; no hydrodynamic convolution step is required.
edam_operation: http://edamontology.org/operation_0282
tools:
- name: plaNETic
  role: Generates the '*_posterior.npy' internal-structure posterior arrays that are
    aggregated and weighted in this skill
  repo: github.com/joannegger/plaNETic
- name: Python
  role: Used to load the .npy posterior arrays and CSV parameters, run the hydrodynamic
    escape model per draw, and compute weighted medians and credible intervals
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:1a9b041b3fd4a0e77fa39c94542a93c23c6204d9e6e99d8af322a89ba0b79b78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - plaNETic
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# posterior-distribution-aggregation

## Summary

Aggregate per-draw posterior samples of planetary internal structure (e.g. plaNETic '*_posterior.npy' arrays of shape (1, n_samples, 28) with 28 structure/composition parameters) into per-planet, per-prior-option summary distributions such as median and credible intervals of upper-atmospheric escape rates and envelope retention.

## When to use

Use when grid/accept-reject sample posteriors exist per planet and prior option (e.g. plaNETic *_posterior.npy files) and you need weighted summary statistics (median, credible intervals) of derived quantities such as mass-loss rates.

## When NOT to use

- The observed radii you condition on are unreliable — the paper warns that TESS-only radii for small planets with low per-transit SNR and large gaps can be biased (planet d deviates >3σ from the discovery paper).
- Planet masses fall outside the 0.5–15 Earth-mass range that the plaNETic neural-network surrogate was trained for.
- Only raw posterior samples are needed without downstream derived quantities — simply load the .npy arrays; no hydrodynamic convolution step is required.

## Inputs

- '*_posterior.npy' files from the plaNETic 'posteriors' subfolder (shape (1, n_samples, 28) per prior option, per planet)
- 'stellar_planetary_parameters.csv' with observed host-star and planetary properties
- Posterior draws of internal structure: core/iron/silicate/water/H-He envelope fractions per planet

## Outputs

- 'upper_atmospheric_properties.csv' with one row per planet and prior option
- Per-planet median and credible intervals of the atmospheric escape rate
- Per-planet median and credible intervals of remaining envelope mass fraction (envelope retention)

## How to apply

Load each '*_posterior.npy' from the 'posteriors' subfolder produced by the plaNETic run_grid executable — each array has shape (1, n_samples, 28) per prior option (A1–A3, B1–B3), reflecting different water-abundance and Si/Mg/Fe priors. Load 'stellar_planetary_parameters.csv' to supply stellar irradiation (XUV flux at the HIP 29442 orbital periods of 13.6, 3.5, and 6.4 days) and mass/radius constraints. Feed each sampled posterior draw — core/iron/silicate/water/H-He envelope fractions — into the hydrodynamic escape model, then convolve the per-draw escape rates with the posterior weighting to obtain posterior distributions of the mass-loss rate and remaining envelope mass fraction. Aggregate into one row per planet and prior option with median and credible intervals, saved as upper_atmospheric_properties.csv.

## Related tools

- **plaNETic** (Generates the '*_posterior.npy' internal-structure posterior arrays that are aggregated and weighted in this skill) — github.com/joannegger/plaNETic
- **Python** (Used to load the .npy posterior arrays and CSV parameters, run the hydrodynamic escape model per draw, and compute weighted medians and credible intervals)

## Examples

```
import numpy as np; post = np.load('run_grid/TOI-469_Egger+/posteriors/planet_b_posterior.npy'); assert post.shape[0] == 1 and post.shape[2] == 28
```

## Evaluation signals

- Output file 'upper_atmospheric_properties.csv' exists and contains entries for planets b, c, and d under all prior options
- Loaded posterior arrays have the expected shape (1, n_samples, 28); e.g. README reference runs yield shapes such as (1, 771909, 28) for TOI-238 b, option A1
- Computed escape rates are qualitatively consistent with the inferred envelope masses of the posterior structures (no quantitative threshold is reported in the source)
- Reported medians are bracketed by the credible intervals, and each planet-prior row contains non-null values
- Envelope-retention conclusions are consistent across the six prior options given the differing posterior sample counts per option

## Limitations

- If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage — the posterior arrays cannot be generated until the networks are supplied by the author.
- Reducing the default sampling (10'000 stars × 10'000 planets per star) shortens runtime but yields smaller posterior distributions, widening credible intervals.
- Validation of escape rates against envelope masses is only qualitative; the source text reports no quantitative consistency threshold.

## Evidence

- [readme] Then simply adapt and run the executable, which will generate 'xxx_posterior.npy' files in a subfolder 'posteriors'.: "Then simply adapt and run the executable, which will generate 'xxx_posterior.npy' files in a subfolder 'posteriors'."
- [abstract] We then applied hydrodynamic models to explore the upper atmospheric properties of these inferred structures: "We then applied hydrodynamic models to explore the upper atmospheric properties of these inferred structures"
- [intro] The code efficiently computes posteriors of a planet's internal structure based on its observed planetary and stellar parameters.: "The code efficiently computes posteriors of a planet's internal structure based on its observed planetary and stellar parameters."
