---
name: halo-assembly-bias-quantification
description: Use when you have per-halo mass accretion histories (or diffmah fits)
  plus halo positions and a large-scale density field from a simulation (e.g. TNG),
  and need to measure assembly bias or validate an assembly-history model's residuals.
when_to_use_negative:
- You only need to fit individual halo accretion histories with diffmah (fitting,
  not bias quantification)
- No large-scale density field or halo positions are available, so environmental correlation
  cannot be computed
- Halos below the 10^11 solar-mass present-day cutoff or times t < 1 Gyr, where diffmah
  accuracy was not demonstrated
edam_operation: http://edamontology.org/operation_0280
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: diffmah
  role: Provides the differentiable 3-parameter accretion-history fits whose residuals
    and assembly classes are correlated with the density field
  repo: https://github.com/ArgonneCPAC/diffmah
- name: numpy
  role: Computes Pearson correlation estimators between assembly class / fit residuals
    and the large-scale density field
- name: JAX
  role: Autodiff backend underlying diffmah used to generate the accretion-history
    fits and residuals
- name: TNG
  role: Simulation dataset supplying halo positions, present-day masses, and the large-scale
    density field
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:699cb2f294ad3e4eb81b8699ec27e6da3fea47946216a7dd70144b955d817c34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - JAX
  - Python
  - TNG
  - diffmah
  - numpy
  edam_operation: http://edamontology.org/operation_0280
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# halo-assembly-bias-quantification

## Summary

Quantifies halo assembly bias by correlating halo assembly classes (fast- vs. slow-accreting) with the large-scale density field, and validates a diffmah-based assembly model by checking that fit residuals are uncorrelated with density.

## When to use

Use when you have per-halo mass accretion histories (or diffmah fits) plus halo positions and a large-scale density field from a simulation (e.g. TNG), and need to measure assembly bias or validate an assembly-history model's residuals.

## When NOT to use

- You only need to fit individual halo accretion histories with diffmah (fitting, not bias quantification)
- No large-scale density field or halo positions are available, so environmental correlation cannot be computed
- Halos below the 10^11 solar-mass present-day cutoff or times t < 1 Gyr, where diffmah accuracy was not demonstrated

## Inputs

- Per-halo diffmah 3-parameter fit parameters of halo mass accretion histories (from cosmological simulation merger trees)
- Per-halo growth-history fit residuals for halos with present-day mass > 10^11 solar masses and t > 1 Gyr
- Halo positions (x, y, z) from the TNG simulation or TNG DMO run
- Large-scale density field mapped to halo environments
- Per-halo present-day masses

## Outputs

- Correlation coefficients between halo assembly class and large-scale density (assembly bias signal)
- Pearson correlation coefficients between diffmah fit residuals and local density
- CSV files such as assembly_bias_correlations.csv and residual_density_correlation.csv

## How to apply

Load per-halo diffmah 3-parameter accretion-history fits (which approximate halo growth to ~0.1 dex for t > 1 Gyr and present-day M > 10^11 solar masses) and load halo positions and the large-scale density field from the TNG simulation with numpy. Assign each halo its local density value, and classify halos into fast- vs. slow-accreting populations based on their fitted assembly histories. Compute Pearson correlation coefficients with numpy: (1) assembly class vs. local density, which measures the assembly bias signal, and (2) diffmah fit residuals vs. local density, which tests whether model approximation errors leak environmental information. Correct application requires the residual correlations to be negligible relative to the assembly bias signal, reproducing the paper's finding that assembly bias is accurately captured; write measurements to CSV (e.g. assembly_bias_correlations.csv, residual_density_correlation.csv).

## Related tools

- **diffmah** (Provides the differentiable 3-parameter accretion-history fits whose residuals and assembly classes are correlated with the density field) — https://github.com/ArgonneCPAC/diffmah
- **numpy** (Computes Pearson correlation estimators between assembly class / fit residuals and the large-scale density field)
- **JAX** (Autodiff backend underlying diffmah used to generate the accretion-history fits and residuals)
- **TNG** (Simulation dataset supplying halo positions, present-day masses, and the large-scale density field)

## Examples

```
conda install -c conda-forge diffmah
```

## Evaluation signals

- Residual-vs-density Pearson correlations are negligible compared to the assembly-bias signal between assembly class and density
- Sample restricted to halos with present-day mass > 10^11 solar masses and t > 1 Gyr, matching the 0.1 dex diffmah accuracy regime
- Assembly class split (fast vs. slow accreters) reproduces a nonzero, physically sensible assembly-bias correlation with density
- Correlation magnitudes written to CSV (residual_density_correlation.csv / assembly_bias_correlations.csv) are consistent with the paper's finding of negligible residual correlation
- Fit residuals are the ones produced by the 3-parameter diffmah model, not another parameterization

## Limitations

- diffmah's 0.1 dex accuracy applies for t > 1 Gyr and present-day halo masses above 10^11 solar masses; outside this regime residual correlations may not be trustworthy
- Validation relies on the TNG hydrodynamical simulation (and gravity-only runs); extrapolation to other simulations or observed galaxy samples is untested here
- Requires a defined large-scale density field and halo positions; without environment mapping the bias/residual decomposition cannot be performed

## Evidence

- [abstract] assembly bias accurately captured, residuals uncorrelated: "the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model, and that residual errors in our approximations to halo"
- [abstract] 3-parameter model accuracy regime: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] JAX autodiff implementation: "Our python implementation is based on the autodiff library JAX"
- [readme] data location for project: [evidence span withheld — anchor status: external_doc]
