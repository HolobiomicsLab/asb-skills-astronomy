---
name: tess-discovery-photometry-radius-comparison
description: Use when a planet's radius has been refit with higher-quality or joint
  photometry (e.g. CHEOPS+TESS) and you need to check whether the revised value is
  statistically inconsistent (>3σ) with the TESS-only discovery value, especially
  for small planets with low per-transit SNR.
when_to_use_negative:
- The planet radius posterior samples are not available and only the discovery-paper
  median radius exists (nothing to compare against)
- The analysis goal is internal-structure inference or formation-history modelling
  rather than validating/refuting a published radius measurement
- The comparison target is not a TESS-only discovery value (e.g. comparing two independent
  high-SNR photometric fits, where the low per-transit SNR caveat does not apply)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: plaNETic
  role: Produces the radius posterior samples (xxx_posterior.npy) that supply the
    derived radius and uncertainty used in the sigma-deviation calculation
  repo: github.com/joannegger/plaNETic
- name: Python
  role: Used with numpy to load and summarize the posterior samples and compute the
    uncertainty-weighted deviation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:ff3fac9fdbf0ffc82d872b0e7ef96cb0ceb5efdb4d2d5fac9147ec3578b4b645
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - plaNETic
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# tess-discovery-photometry-radius-comparison

## Summary

Quantifies the uncertainty-weighted deviation between a revised planet radius (derived from joint CHEOPS+TESS photometry) and the median radius from a TESS-only discovery paper, to test whether TESS photometry alone yields biased radii for small planets. Applied here to HIP 29442 (TOI-469) planet d, whose 1.538 ± 0.049 R⊕ radius deviates by more than 3σ from the discovery-paper median.

## When to use

Use when a planet's radius has been refit with higher-quality or joint photometry (e.g. CHEOPS+TESS) and you need to check whether the revised value is statistically inconsistent (>3σ) with the TESS-only discovery value, especially for small planets with low per-transit SNR.

## When NOT to use

- The planet radius posterior samples are not available and only the discovery-paper median radius exists (nothing to compare against)
- The analysis goal is internal-structure inference or formation-history modelling rather than validating/refuting a published radius measurement
- The comparison target is not a TESS-only discovery value (e.g. comparing two independent high-SNR photometric fits, where the low per-transit SNR caveat does not apply)

## Inputs

- Planet radius posterior samples (.npy file, e.g. xxx_posterior.npy from the 'posteriors' subfolder of a plaNETic run_grid)
- Median radius value from the TESS-only discovery paper for the target planet
- Derived radius and uncertainty from the joint CHEOPS+TESS fit (e.g. 1.538 ± 0.049 R⊕ for HIP 29442 d)
- 3σ significance threshold

## Outputs

- sigma_deviation_planet_d.csv containing the computed sigma deviation and the threshold comparison
- Boolean/qualitative verdict on whether the deviation exceeds 3σ

## How to apply

Load the planet radius posterior samples (xxx_posterior.npy) from the 'posteriors' subfolder of the plaNETic run grid using numpy in a Python 3.9 environment. Summarize the posterior to recover the derived radius and its uncertainty (for HIP 29442 d: 1.538 ± 0.049 R⊕ from joint modelling of 9.6 days of CHEOPS data with two TESS sectors). Retrieve the median radius reported in the TESS-only discovery paper and compute the uncertainty-weighted difference (derived minus median, divided by the combined uncertainty) to express the deviation in sigma. Compare the computed deviation against a 3σ significance threshold and record both the value and the threshold comparison in sigma_deviation_planet_d.csv. The rationale is that TESS photometry with low per-transit SNR and large gaps between observations can bias radii of small planets, so a >3σ deviation flags the discovery value as unreliable.

## Related tools

- **plaNETic** (Produces the radius posterior samples (xxx_posterior.npy) that supply the derived radius and uncertainty used in the sigma-deviation calculation) — github.com/joannegger/plaNETic
- **Python** (Used with numpy to load and summarize the posterior samples and compute the uncertainty-weighted deviation)

## Examples

```
import numpy as np; post = np.load('run_grid/TOI-469_Egger+/posteriors/planet_d_posterior.npy'); r, u = post_summary(post); sigma = (r - r_disc) / np.hypot(u, u_disc)
```

## Evaluation signals

- The computed deviation for HIP 29442 d must exceed 3σ, matching the paper's reported finding that the value 'deviates by more than 3σ from the median value reported in the discovery paper'
- The derived radius recovered from the posterior summary matches the published value of 1.538 ± 0.049 R⊕ for planet d
- sigma_deviation_planet_d.csv exists and contains both the numeric deviation and the threshold comparison
- The posterior samples were loaded from the 'posteriors' subfolder (shape consistent with plaNETic output, e.g. (1, N, 28) arrays)

## Limitations

- The >3σ caution conclusion applies specifically to TESS photometry used for small planets with low per-transit SNR and large gaps between observations; it does not automatically generalize to other instruments or high-SNR planets
- plaNETic requires access to a GPU to run efficiently; CPU-only runs drastically increase computation time and are not recommended
- If plaNETic cannot find the trained DNNs, this is an issue with GitHub's Large File Storage and the authors must be contacted
- The derived radius depends on the joint CHEOPS+TESS modelling setup (9.6 days CHEOPS, two TESS sectors) and prior choices (water abundance inside vs. outside iceline, Si/Mg/Fe ratio priors), so deviations reflect those modelling assumptions

## Evidence

- [abstract] >3σ deviation finding: "For planet d this value deviates by more than 3σ from the median value reported in the discovery paper, leading us to conclude that caution is required when using TESS photometry"
- [abstract] derived radius value: "planetary radii of 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, and d, which orbit HIP 29442 with periods of 13.6, 3.5, and 6.4 days"
- [abstract] photometry basis: "We observed HIP 29442 with CHEOPS for a total of 9.6 days, which we modelled jointly with two sectors of TESS data to derive planetary radii"
- [methods] posterior file generation: "Then simply adapt and run the executable, which will generate 'xxx_posterior.npy' files in a subfolder 'posteriors'."
- [readme] GPU requirement: "plaNETic needs access to a GPU to run efficiently. While it is in principle possible to run the code on CPUs only, this will drastically increase the computation time and is not recommended."
