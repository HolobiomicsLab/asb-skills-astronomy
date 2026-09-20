---
name: interaction-term-model-formulation
description: Use when building a photometric redshift regression from PCA-reduced
  galaxy magnitudes where a simple additive formula fails to capture the data's complexity
  and interaction terms among PCs are required.
when_to_use_negative:
- When the exploratory model uses only main-effect (additive) PC terms — the paper
  warns that 'tests using a simple formulation without interaction terms are inadequate
  in capturing the complexity of the data', so an additive-only formulation is the
  known-wrong choice here
- When the response distribution is not compatible with a gamma family with log link
  — the paper notes the default 'gamma family and inverse power link may not be fully
  optimal in all situations', so non-positive, non-skewed responses need a different
  family
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Fits the gamma-family GLM with the factorial formula and interaction terms
    via iteratively reweighted least squares (glm() IRLS)
- name: Python
  role: Alternative implementation platform for the PCA + gamma GLM photo-z fit (statsmodels-backed)
  repo: https://github.com/statsmodels
- name: CosmoPhotoz
  role: Released R/Python/Shiny package implementing the GLM photo-z estimation with
    the interaction-term factorial formula
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:f232e0deeb15154ec9278caf6d21d337768d15699a339880ac6eebf84ddd76fb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CosmoPhotoz
  - Python
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# interaction-term-model-formulation

## Summary

Formulate the GLM photo-z model as a factorial formula zphot ~ PC1 * PC2 * ... * PCn + C, where the crossing operator generates interaction terms among the robust-PCA principal components, then fit it with a gamma family and log link via iteratively reweighted least squares. Interaction terms are essential: the paper's authors found simpler additive formulations inadequate for the PHAT0 magnitude data.

## When to use

Use when building a photometric redshift regression from PCA-reduced galaxy magnitudes where a simple additive formula fails to capture the data's complexity and interaction terms among PCs are required.

## When NOT to use

- When the exploratory model uses only main-effect (additive) PC terms — the paper warns that 'tests using a simple formulation without interaction terms are inadequate in capturing the complexity of the data', so an additive-only formulation is the known-wrong choice here
- When the response distribution is not compatible with a gamma family with log link — the paper notes the default 'gamma family and inverse power link may not be fully optimal in all situations', so non-positive, non-skewed responses need a different family

## Inputs

- Principal component projections (PC1..PCn) of galaxy magnitudes from robust PCA of the complete training+test data set
- Training catalogue with spectroscopic redshifts (e.g., PHAT0train with 161,042 galaxies, or the paper's 33,904-galaxy 1:9 training split)
- Number of PCs selected at the ~99.5% cumulative variance threshold (6 for PHAT0)

## Outputs

- Best-fit gamma-family GLM (log link) with factorial formula and interaction terms
- Predicted photo-z values for the test set (e.g., 8,478 PHAT0test objects)
- Photo-z quality metrics: bias, rms(delta-z), catastrophic outlier rate (reported ~0.033, ~0.025, ~1.367% for PHAT0)

## How to apply

After running robust PCA on the complete (training + test) data and retaining the ~99.5% cumulative-variance threshold of PCs (6 components for PHAT0), express the model with the factorial (crossing) formula zphot ~ PC1 * PC2 * ... * PCn + C so that interaction terms among the PCs are included. Fit the formula with a gamma family distribution and log link using iteratively reweighted least squares on the training catalogue (e.g., the 33,904-galaxy PHAT0 training split or the CosmoPhotoz PHAT0train set). Do not fall back to a formulation without interaction terms — the paper explicitly notes such tests are inadequate for capturing the complexity of the data. Use the best-fit GLM with the test-set PC projections to predict photo-zs, then evaluate bias, rms(delta-z), and the catastrophic outlier fraction (|z_phot - z_spec|/(1 + z_spec) > 0.10).

## Related tools

- **R** (Fits the gamma-family GLM with the factorial formula and interaction terms via iteratively reweighted least squares (glm() IRLS))
- **Python** (Alternative implementation platform for the PCA + gamma GLM photo-z fit (statsmodels-backed)) — https://github.com/statsmodels
- **CosmoPhotoz** (Released R/Python/Shiny package implementing the GLM photo-z estimation with the interaction-term factorial formula) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
glm(zphot ~ (PC1*PC2*PC3*PC4*PC5*PC6) + C, family = Gamma(link = "log"), data = PHAT0train)
```

## Evaluation signals

- The fitted formula string uses the crossing operator (*) among all retained PCs (PC1..PC6 for PHAT0) plus the C term, so interaction terms are present in the model frame
- Predicted metrics on the test set reproduce the reported PHAT0 values within tolerance: bias ~0.033, rms(delta-z) ~0.025, catastrophic outlier rate ~1.367% (~1.4%)
- Catastrophic outlier rate stays near the paper's reported ~1% level for simulated data, computed as |z_phot - z_spec|/(1 + z_spec) > 0.10
- Number of PCs entering the formula matches the ~99.5% cumulative variance threshold (at least 6 components for PHAT0)
- Output CSV (phat0_glm_metrics.csv) contains the three named metric fields: bias, rms, catastrophic outlier rate

## Limitations

- Tests using a simple formulation without interaction terms are inadequate in capturing the complexity of the data — dropping interactions degrades the fit
- Slight differences exist between the R and Python implementations in how they solve the SVD matrices in the PCA step and how they minimise the GLM, so cross-language results may differ slightly
- The default gamma family and inverse power link may not be fully optimal in all situations; the log-link factorial formulation is dataset-specific

## Evidence

- [methods] IRLS fitting of the factorial formula: [evidence span withheld — anchor status: not_found]
