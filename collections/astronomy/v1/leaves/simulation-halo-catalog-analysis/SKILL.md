---
name: simulation-halo-catalog-analysis
description: Use when a task requires connecting halo assembly histories from a simulated
  halo catalog (e.g., TNG) to large-scale environment density, or quantifying assembly
  bias and model residual correlations.
when_to_use_negative:
- The analysis does not involve halo assembly histories or environment correlations
  (e.g., pure galaxy-halo connection fits with no density-field data).
- Halo masses fall below the 10^11 solar-mass present-day threshold or the time range
  of interest extends below t > 1 Gyr, where the 0.1-dex diffmah accuracy does not
  hold.
- Data come from observations rather than a simulated halo catalog with merger trees
  and a large-scale density field (e.g., TNG).
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: diffmah
  role: Differentiable 3-parameter model providing halo mass accretion-history fits
    used to classify halos as fast- vs. slow-accreting and to compute approximation
    residuals
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: numpy
  role: Loads halo positions and density field from the TNG simulation and computes
    the numpy-based correlation estimators
- name: jax
  role: Autodiff backend underlying the diffmah model implementation
  repo: https://jax.readthedocs.io/en/latest/
- name: diffmah
  role: Supplies the halo-fitting workflow (diffmah_fitter_demo.ipynb notebook; history_fitting_script.py
    for parallel MAH fitting with mpi4py)
  repo: https://github.com/ArgonneCPAC/diffmah/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:6518d5e53e97f439b24c96064905ac911717de46d0313dc822fb8f8e92adcd47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - Python
  - diffmah
  - jax
  - numpy
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# simulation-halo-catalog-analysis

## Summary

Analyze halo catalogs from cosmological simulations (e.g., TNG) by loading merger-tree mass accretion histories, diffmah 3-parameter fits, and halo positions, then measuring correlations between halo assembly class and the large-scale density field to quantify halo assembly bias.

## When to use

Use when a task requires connecting halo assembly histories from a simulated halo catalog (e.g., TNG) to large-scale environment density, or quantifying assembly bias and model residual correlations.

## When NOT to use

- The analysis does not involve halo assembly histories or environment correlations (e.g., pure galaxy-halo connection fits with no density-field data).
- Halo masses fall below the 10^11 solar-mass present-day threshold or the time range of interest extends below t > 1 Gyr, where the 0.1-dex diffmah accuracy does not hold.
- Data come from observations rather than a simulated halo catalog with merger trees and a large-scale density field (e.g., TNG).

## Inputs

- diffmah 3-parameter accretion-history fits for halos (from fitting TNG merger trees)
- Halo positions from the TNG simulation
- Large-scale density field from the TNG simulation
- Halo merger trees from a high-resolution cosmological simulation (for refitting MAHs)

## Outputs

- Correlation measurements between halo assembly class (fast vs. slow accretion) and the large-scale density field
- Correlation measurements between diffmah approximation residuals and the density field
- assembly_bias_correlations.csv containing the correlation measurements

## How to apply

Load the diffmah 3-parameter accretion-history fits (which approximate halo growth to ~0.1 dex accuracy for t > 1 Gyr and present-day halo masses above 10^11 solar masses) alongside halo positions and the large-scale density field from the TNG simulation using numpy. Assign each halo its large-scale density environment, then classify halos into fast- vs. slow-accreting populations based on their fitted assembly histories. With numpy-based correlation estimators, compute (1) the correlation between halo assembly class and the large-scale density field, and separately (2) the correlation between diffmah approximation residuals and the density field. The decision point is the validation check: residual correlations between diffmah approximation errors and the density field must be negligible to reproduce the paper's finding that assembly bias is accurately captured by the model; write the measurements to a CSV such as assembly_bias_correlations.csv. Fitting of the MAHs themselves can be done with the diffmah fitter, in parallel with mpi4py for large samples of merger trees.

## Related tools

- **diffmah** (Differentiable 3-parameter model providing halo mass accretion-history fits used to classify halos as fast- vs. slow-accreting and to compute approximation residuals) — https://github.com/ArgonneCPAC/diffmah/
- **numpy** (Loads halo positions and density field from the TNG simulation and computes the numpy-based correlation estimators)
- **jax** (Autodiff backend underlying the diffmah model implementation) — https://jax.readthedocs.io/en/latest/
- **diffmah** (Supplies the halo-fitting workflow (diffmah_fitter_demo.ipynb notebook; history_fitting_script.py for parallel MAH fitting with mpi4py)) — https://github.com/ArgonneCPAC/diffmah/

## Examples

```
conda create -c conda-forge -n diffit python=3.9 numpy jax pytest ipython jupyter matplotlib scipy h5py diffmah
```

## Evaluation signals

- Residual correlations between diffmah approximation errors and the density field are negligible (near-zero), matching the paper's finding that assembly bias is accurately captured.
- Measured assembly-bias signal between fast/slow accreting classes and the density field is non-negligible and consistent with the paper's result.
- Halo sample respects the validity thresholds: present-day mass > 10^11 solar masses and t > 1 Gyr, with ~0.1 dex typical diffmah fit accuracy.
- Correlation measurements are successfully written to assembly_bias_correlations.csv with the expected structure.
- Fitted MAHs reproduce the mean and variance of halo mass accretion rate across cosmic time, as claimed for the diffmah model.

## Limitations

- The 0.1-dex approximation accuracy of the diffmah 3-parameter model holds only for t > 1 Gyr and halos with present-day mass greater than 10^11 solar masses.
- diffmah requires python >= 3.9 and depends on numpy and jax; environment must be configured accordingly (e.g., conda create -c conda-forge -n diffit python=3.9 numpy jax ...).
- Project data for the diffmah workflows is hosted externally at portal.nersc.gov (NERSC portal), which must be accessible.
- The article's discussion section content was absent in the extracted metadata, limiting detail on repository status (no changelog found).

## Evidence

- [abstract] the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model: "the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model"
- [abstract] 3-parameter model approximates halo growth with 0.1 dex accuracy for t > 1 Gyr and M > 10^11 Msun: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] Using large samples of halo merger trees taken from high-resolution cosmological simulations: "Using large samples of halo merger trees taken from high-resolution cosmological simulations"
- [methods] diffmah requires python>=3.9, and has the following dependencies numpy, jax: "diffmah requires python>=3.9, and has the following dependencies"
- [readme] README: history_fitting_script.py example for parallel MAH fitting with mpi4py: [evidence span withheld — anchor status: external_doc]
