---
name: posterior-credible-region-visualization
description: Use when you have converged MCMC posterior samples (e.g. from emcee over
  a 2-parameter BBN/NP likelihood) and need joint 68%/95% credible regions as a corner
  plot plus an upper bound value on a physical parameter.
when_to_use_negative:
- Input is a single best-fit point or point estimate without a sampled posterior —
  credible regions cannot be computed without posterior samples
- The chain has not passed burn-in/convergence checks — contours from a pre-convergence
  chain would be misleading
- The analysis requires frequentist confidence intervals rather than Bayesian credible
  regions from posterior samples
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2269
tools:
- name: PRyMordial
  role: supplies the BBN-predicted observables (Y_p, D/H) whose likelihood feeds the
    posterior that is visualized; its np/SM parameters define the corner-plot axes
  repo: github.com/vallima/PRyMordial
- name: emcee
  role: affine-invariant ensemble sampler producing the posterior chain whose 68%/95%
    credible contours are rendered
- name: matplotlib
  role: renders the corner plot of joint credible regions from the posterior samples
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:645d46fde1eebb250fbca4aa4d190b8cdbd866ec788f400d7df22646b91f0650
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRyMordial
  - Python 3
  - emcee
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# posterior-credible-region-visualization

## Summary

Visualize Bayesian posterior samples from an MCMC analysis (e.g. emcee runs over PRyMordial BBN observables) as joint 68%/95% credible contours in a matplotlib corner plot, and extract credible bounds on the parameter of interest such as NP_delta_nTOp.

## When to use

Use when you have converged MCMC posterior samples (e.g. from emcee over a 2-parameter BBN/NP likelihood) and need joint 68%/95% credible regions as a corner plot plus an upper bound value on a physical parameter.

## When NOT to use

- Input is a single best-fit point or point estimate without a sampled posterior — credible regions cannot be computed without posterior samples
- The chain has not passed burn-in/convergence checks — contours from a pre-convergence chain would be misleading
- The analysis requires frequentist confidence intervals rather than Bayesian credible regions from posterior samples

## Inputs

- converged emcee posterior chain (flattened samples array over parameters such as NP_delta_nTOp and nuisance priors on tau_n, Omega_B h^2)
-  credible levels to plot (68% and 95%)
-  parameter labels/ranges for the corner plot

## Outputs

- matplotlib corner plot showing joint 68% and 95% credible contours in the parameter plane
- marginal posterior histograms with 68%/95% credible intervals
- upper credible bound value on the constrained parameter (e.g. few-percent NP Born-rate modification limit)
- saved posterior chain file and plot file on disk

## How to apply

After the emcee affine-invariant ensemble sampler has converged on the posterior (e.g. NP_delta_nTOp with Gaussian priors on tau_n and Omega_B h^2 from PDG measurements), take the flattened chain of posterior samples and bin them to compute the joint 68% and 95% credible contours. Render these contours as a corner plot with matplotlib, overlaying the two credible levels in the parameter plane. Extract the marginal posterior for the constrained parameter (e.g. NP_delta_nTOp) and report the upper credible bound; in the PRyMordial context this should translate into a few-percent upper limit on the NP modification of the n_TOp Born rates. Save the chain, the corner plot, and the bound value to disk so the posterior and its visualization remain jointly traceable to the validation requirement that the reproduced contours match the paper's reported regions.

## Related tools

- **PRyMordial** (supplies the BBN-predicted observables (Y_p, D/H) whose likelihood feeds the posterior that is visualized; its np/SM parameters define the corner-plot axes) — github.com/vallima/PRyMordial
- **emcee** (affine-invariant ensemble sampler producing the posterior chain whose 68%/95% credible contours are rendered)
- **matplotlib** (renders the corner plot of joint credible regions from the posterior samples)

## Examples

```
python runPRyM_julia.py  # quick example; posterior chain then corner-plot: corner.corner(flat_samples, levels=[0.68, 0.95])
```

## Evaluation signals

- The reproduced 68% and 95% joint credible regions visually and numerically match the paper's reported contours for NP_delta_nTOp
- The extracted upper bound on the NP modification of the n_TOp Born rates remains at the few-percent level, consistent with the reported finding
- Contour levels are computed from the same flattened chain used for the bound, so plot and bound agree on the same posterior
- The corner plot, posterior chain, and upper-bound value are all saved to disk and reproducible from the same run
- Marginal credible intervals in the plot are consistent with the joint contours at the corresponding credibility levels

## Limitations

- Requires converged posterior samples — the workflow mandates 'enough walkers and burn-in steps to converge the 2-parameter posterior' before visualizing
- Contours depend on prior choices (Gaussian priors on tau_n and Omega_B h^2 from PDG); changing priors changes the credible regions
- The bound is only as reliable as the underlying PRyMordial rate configuration (smallnet_flag, nacreii_flags) used in the likelihood

## Evidence

- [methods] Compute the joint 68% and 95% credible contours from the posterior samples and render them as a corner plot with matplotlib.: [evidence span withheld — anchor status: not_found]
- [methods] emcee affine-invariant sampler convergence requirement: [evidence span withheld — anchor status: not_found]
- [results] BBN consistency at few-percent NP level: [evidence span withheld — anchor status: not_found]
- [methods] Gaussian priors definition: [evidence span withheld — anchor status: not_found]
