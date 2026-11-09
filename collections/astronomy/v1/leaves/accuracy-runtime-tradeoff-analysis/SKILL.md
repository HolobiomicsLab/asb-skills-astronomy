---
name: accuracy-runtime-tradeoff-analysis
description: Decision support for choosing the number of PCA explanatory variables
  in a gamma-GLM photo-z pipeline when both catastrophic error (|z_phot - z_spec|
  > 0.10) and compute budget matter.
when_to_use_negative:
- The goal is a single production photo-z fit with a fixed, pre-validated number of
  components rather than an explicit accuracy-vs-runtime comparison
- Simple GLM formulations without interaction terms are adequate — the paper notes
  such tests 'are inadequate in capturing the complexity of the data', so sweeps of
  the factorial formula would be misleading if the formula is simplified
- Compute timing results are needed for hardware far different from the reference
  low-end laptop (AMD Athlon X2, 1.7 GB RAM, Ubuntu 10.04), since absolute times do
  not transfer
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: R
  role: Fits the gamma-family GLM with log link and factorial interaction formula
    via iteratively reweighted least squares for each PC configuration; timing is
    recorded in R
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: Python
  role: Alternative implementation of the CosmoPhotoz pipeline (PCA + GLM) used to
    sweep PC counts and predict photo-z on test data
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: CosmoPhotoz
  role: Public package (R, R/Shiny, Python) implementing the GLM photo-z estimation
    used to run the PC sweep
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:3c55178d4f904b11d481fdb18a4532a7b4fb016df127cfe5440b2fa34d025228
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# accuracy-runtime-tradeoff-analysis

## Summary

Quantifies how model complexity (number of principal components) trades catastrophic outlier rate against GLM fitting wall-clock time, as demonstrated on the PHAT0 photometric-redshift gamma-family GLM (log link) swept over 6, 8, and 10 PCs.

## When to use

Decision support for choosing the number of PCA explanatory variables in a gamma-GLM photo-z pipeline when both catastrophic error (|z_phot - z_spec| > 0.10) and compute budget matter.

## When NOT to use

- The goal is a single production photo-z fit with a fixed, pre-validated number of components rather than an explicit accuracy-vs-runtime comparison
- Simple GLM formulations without interaction terms are adequate — the paper notes such tests 'are inadequate in capturing the complexity of the data', so sweeps of the factorial formula would be misleading if the formula is simplified
- Compute timing results are needed for hardware far different from the reference low-end laptop (AMD Athlon X2, 1.7 GB RAM, Ubuntu 10.04), since absolute times do not transfer

## Inputs

- Photometric catalogue with magnitudes and spectroscopic redshifts (e.g., PHAT0 simulated catalogue)
- Random 1:9 training/test split of the catalogue (33,904 / 135,616 galaxies for PHAT0)
- Robust PCA decomposition of the full (training + test) magnitude set
- Candidate PC counts determined by the ~99.5% cumulative-variance threshold

## Outputs

- Catastrophic outlier rate O(a) per PC configuration (threshold Delta_z > 0.10)
- Wall-clock GLM fitting time per PC configuration (s)
- Outlier-rate-versus-fitting-time trade-off table (e.g., 4.4%/170 s, 2.5%/1200 s, 1.4%/4948 s)

## How to apply

Split the catalogue into training and test sets at a 1:9 ratio (PHAT0: 33,904 training, 135,616 test galaxies), then run robust PCA on the complete data set and select PCs using a cumulative variance threshold of ~99.5% to bracket the candidate counts (PHAT0 needed at least 6 PCs). For each n in {6, 8, 10}, fit a gamma-family GLM with a log link using the factorial R formula with interaction terms (e.g., PC1*PC2*...*PCn) via iteratively reweighted least squares, and record the wall-clock fitting time. Predict photo-z for the test set from the PC projections and each best-fit GLM, computing the catastrophic outlier rate O(a) with Delta_z(a) = |z_phot - z_spec| > 0.10. Compile an outlier-rate-versus-fitting-time table and pick the configuration meeting the accuracy requirement within the available runtime; the reference points are 4.4%/170 s (6 PCs), 2.5%/1200 s (8 PCs) and 1.4%/4948 s (10 PCs).

## Related tools

- **R** (Fits the gamma-family GLM with log link and factorial interaction formula via iteratively reweighted least squares for each PC configuration; timing is recorded in R) — https://github.com/COINtoolbox/COSMOPhotoz/
- **Python** (Alternative implementation of the CosmoPhotoz pipeline (PCA + GLM) used to sweep PC counts and predict photo-z on test data) — https://github.com/COINtoolbox/COSMOPhotoz/
- **CosmoPhotoz** (Public package (R, R/Shiny, Python) implementing the GLM photo-z estimation used to run the PC sweep) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
R> fit <- glm(z ~ PC1*PC2*PC3*PC4*PC5*PC6, family=Gamma(link='log'), data=train); system.time(p <- predict(fit, newdata=testPC))
```

## Evaluation signals

- Reproduced table matches catastrophic outlier rates of 4.4% (6 PCs), 2.5% (8 PCs) and 1.4% (10 PCs) as reported in section 5.3 of the paper
- Reproduced fitting times are approximately 170 s, 1200 s and 4948 s for 6, 8 and 10 PCs respectively (modulo hardware differences)
- Outlier rate decreases monotonically and fitting time increases monotonically as PCs are added from 6 to 10
- Split sizes match the 1:9 ratio: 33,904 training and 135,616 test galaxies for PHAT0
- PCA retains at least 99.5% of the variance for the smallest configuration (at least 6 components)

## Limitations

- Absolute runtimes are tied to a specific low-end hardware/OS configuration (AMD Athlon X2 Dual-Core QL-64, 1.7 GB RAM, Ubuntu 10.04) and will not transfer to other machines
- Slight differences exist between the R and Python implementations in SVD solving (PCA step) and GLM minimisation, so the two platforms may not give identical trade-off points
- Runtime and accuracy coupling is empirical for this gamma-GLM/log-link setup; the default gamma family and link 'may not be fully optimal in all situations'

## Evidence

- [results] Fitting PHAT0 with 6 PCs yields O(a) = 4.4% catastrophic errors in 170 s; 8 PCs yields 2.5% in 1200 s; 10 PCs yields 1.4% in 4948 s: "Decomposition by PCA showed that at least 6 components needed to be used to ensure that 99.5% of the variance was retained"
- [results] PC selection threshold and split sizes: "We split the data into training and testing with a ratio of 1:9, respectively. This criterion resulted in a training size of 33,904 galaxies and a testing set of 135,616 galaxies"
- [methods] Outlier metric definition: [evidence span withheld — anchor status: not_found]
- [methods] Fitting procedure: [evidence span withheld — anchor status: not_found]
- [discussion] Hardware caveat on timing: "using a standard laptop (AMD Athlon X2 Dual-Core QL-64 processor with 1.7 GB RAM on the Ubuntu 10.04 OS)"
