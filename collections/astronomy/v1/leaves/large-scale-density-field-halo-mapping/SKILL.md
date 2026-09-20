---
name: large-scale-density-field-halo-mapping
description: Use when you have per-halo diffmah growth-history fit residuals and need
  to test whether those residual errors correlate with the large-scale density field,
  i.e. to validate that halo assembly bias is captured by the model.
when_to_use_negative:
- You are still fitting the halo mass accretion histories themselves (producing the
  residuals) rather than testing residual correlations — that upstream diffmah fitting
  is a separate skill.
- Your halo sample falls outside the validated regime, e.g. present-day masses below
  1e11 solar masses or times t <= 1 Gyr, where the paper's accuracy claims do not
  hold.
- The residuals and the density field come from different parent simulations or incompatible
  halo catalogs, so mapping halos to local density would be meaningless.
tools:
- name: numpy
  role: Computes the Pearson correlation coefficients between diffmah fit errors and
    the halo's mapped local density value
- name: JAX
  role: Autodiff library underlying the diffmah model that produced the per-halo growth-history
    residuals being validated
  repo: https://github.com/ArgonneCPAC/diffmah/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:f285f654ed92949f9dfa0531d1f728442b0933dda567c9f58d583b4517814f07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - JAX
  - Python
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# large-scale-density-field-halo-mapping

## Summary

Validation skill from the diffmah paper (Hearin et al. 2021, OJAp) that maps halos onto a large-scale density field and computes Pearson correlations between diffmah fit residuals and local density, to confirm that assembly bias is captured by the model rather than leaked into approximation errors.

## When to use

Use when you have per-halo diffmah growth-history fit residuals and need to test whether those residual errors correlate with the large-scale density field, i.e. to validate that halo assembly bias is captured by the model.

## When NOT to use

- You are still fitting the halo mass accretion histories themselves (producing the residuals) rather than testing residual correlations — that upstream diffmah fitting is a separate skill.
- Your halo sample falls outside the validated regime, e.g. present-day masses below 1e11 solar masses or times t <= 1 Gyr, where the paper's accuracy claims do not hold.
- The residuals and the density field come from different parent simulations or incompatible halo catalogs, so mapping halos to local density would be meaningless.

## Inputs

- Per-halo diffmah growth-history fit residuals (from fitting halo mass accretion histories in the gravity-only simulation; halos with present-day mass > 1e11 solar masses, t > 1 Gyr)
- Halo positions from the same halo population
- Large-scale density field from the public TNG dark-matter-only (DMO) simulation

## Outputs

- Pearson residual-vs-density correlation coefficients over the halo population
- residual_density_correlation.csv containing the recorded correlation magnitudes

## How to apply

Load the per-halo growth-history residuals produced by the diffmah fitting step for halos in the gravity-only simulation, restricted to present-day mass greater than 1e11 solar masses and times t > 1 Gyr — the regime where the paper demonstrates the 3-parameter power-law model reaches ~0.1 dex accuracy. Load halo positions and the large-scale density field from the public TNG dark-matter-only (DMO) run and map each halo to its local density value by its position. With numpy, compute Pearson correlation coefficients of fit errors against local density, evaluated over the whole halo population. Record the correlation magnitudes and judge them against the paper's expectation: residual errors should show negligible correlation with the density field while assembly bias itself is accurately captured. Write the resulting correlation values to a CSV for downstream comparison.

## Related tools

- **numpy** (Computes the Pearson correlation coefficients between diffmah fit errors and the halo's mapped local density value)
- **JAX** (Autodiff library underlying the diffmah model that produced the per-halo growth-history residuals being validated) — https://github.com/ArgonneCPAC/diffmah/

## Examples

```
import numpy as np; r = np.corrcoef(fit_errors, halo_local_density)[0, 1]
```

## Evaluation signals

- Computed correlation magnitudes between fit errors and local density are negligible, consistent with the paper's finding that assembly bias is accurately captured.
- Halo sample respects the paper's cutoffs: present-day mass > 1e11 solar masses and t > 1 Gyr.
- Every halo in the residual file is mapped to exactly one local density value from the TNG DMO field before correlations are computed.
- Pearson coefficients are evaluated over the full halo population, not per-halo or on subsamples.
- Output file residual_density_correlation.csv exists and contains the recorded correlation values.

## Limitations

- The negligible-residual-correlation result is demonstrated for halos of present-day mass greater than 1e11 solar masses and for t > 1 Gyr; smaller masses or earlier times are outside the paper's validation.
- The skill assesses population-level (Pearson) correlation between residuals and density field only; it does not re-fit diffmah parameters or measure assembly bias signal strength directly.
- The article's discussion section content is absent (only repository source metadata and synthesis timestamp are available), so discussion-level guidance on interpreting the correlations is not available.

## Evidence

- [abstract] abstract: "the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model, and that residual errors in our approximations to halo"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [methods] methods: "diffmah requires python>=3.9, and has the following dependencies"
