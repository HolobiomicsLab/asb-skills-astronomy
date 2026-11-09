---
name: kilonova-lightcurve-inference
description: Use when multi-band kilonova photometry (e.g., AT2017gfo) must be fitted
  with an NMMA kilonova model to obtain ejecta-parameter posteriors, or when a GW+kilonova
  joint analysis needs the kilonova likelihood component.
when_to_use_negative:
- Input is GRB afterglow light-curve data alone — use the synthetic Gaussian jet-model
  analysis instead of a kilonova model.
- You need to forward-simulate a light curve from known parameters rather than infer
  parameters from data — use the lightcurve-simulation tutorial / lightcurve-generation
  workflow instead of inference.
- Input is gravitational-wave strain only with no EM counterpart — a pure GW parameter-estimation
  path suffices.
- 'edam_operation": "'
- 'edam_topics": [],  '
tools:
- name: NMMA
  role: Provides the lightcurve-analysis, lightcurve-generation and related entry
    points (nmma-generation, nmma-analysis, create-svdmodel) used to build the kilonova
    likelihood and run the inference
  repo: github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Inference backend through which the kilonova light-curve analysis likelihood
    and sampler are wired
- name: dynesty
  role: Nested sampler used to obtain the kilonova/GW+KN posterior samples
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:86bf94acd3b95b19dfb3e2cf25d7136959f3efb03f6028014cda3d429eee4d55
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NMMA
  - bilby
  - dynesty
  - lightcurve-analysis
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# kilonova-lightcurve-inference

## Summary

Bayesian inference of kilonova light-curve parameters (e.g., dynamical and wind ejecta masses) from multi-band photometry such as AT2017gfo, using NMMA's lightcurve-analysis entry point backed by bilby and nested sampling with dynesty. It is the EM-leg inference step of the NMMA simultaneous GW+KN+GRB analysis that yields R_1.4 = 11.98 (+0.35/−0.40) km.

## When to use

Use when multi-band kilonova photometry (e.g., AT2017gfo) must be fitted with an NMMA kilonova model to obtain ejecta-parameter posteriors, or when a GW+kilonova joint analysis needs the kilonova likelihood component.

## When NOT to use

- Input is GRB afterglow light-curve data alone — use the synthetic Gaussian jet-model analysis instead of a kilonova model.
- You need to forward-simulate a light curve from known parameters rather than infer parameters from data — use the lightcurve-simulation tutorial / lightcurve-generation workflow instead of inference.
- Input is gravitational-wave strain only with no EM counterpart — a pure GW parameter-estimation path suffices.
- edam_operation": "
- edam_topics": [],  

## Inputs

- AT2017gfo multi-band light-curve photometry (kilonova observations)
- NMMA kilonova model (e.g., SVD-model light-curve interpolant; create-svdmodel)
- EOS prior sets (chiral-EFT-based) when sampling radius/EOS parameters jointly
- posterior_samples.hdf5 from a joint GW170817 + AT2017gfo + GRB170817A analysis (for best-fit light-curve generation)

## Outputs

- Posterior samples for kilonova ejecta parameters (dynamical and wind ejecta masses), R_1.4, and inclination (posterior_samples.hdf5 / gw_kn_only_posterior_samples.json)
- Best-fit kilonova light curve with credible band (e.g., 1-magnitude uncertainty band as in Fig. 2)
- Comparison plot of best-fit light curves overlaid on observed data (e.g., bestfit_lightcurves.png)

## How to apply

Generate the kilonova likelihood products and SVD-model interpolants for the chosen kilonova model, then invoke NMMA's `lightcurve-analysis` entry point on the AT2017gfo multi-band photometry with bilby as the inference backend and the dynesty nested sampler. Sample the joint posterior over kilonova ejecta parameters (dynamical and wind ejecta masses) and, in the joint setting, tie the analysis to the GW and GRB afterglow likelihoods via `nmma-analysis`. To produce best-fit curves after sampling, evaluate the kilonova model at posterior parameter values from `posterior_samples.hdf5` using the `lightcurve-generation` entry point as shown in tutorials/tutorial-lightcurve_simulation.ipynb. Overlay the best-fit light curve and its credible band (e.g., the 1-magnitude band of Fig. 2) on the observed multi-band data to validate the fit.

## Related tools

- **NMMA** (Provides the lightcurve-analysis, lightcurve-generation and related entry points (nmma-generation, nmma-analysis, create-svdmodel) used to build the kilonova likelihood and run the inference) — github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Inference backend through which the kilonova light-curve analysis likelihood and sampler are wired)
- **dynesty** (Nested sampler used to obtain the kilonova/GW+KN posterior samples)

## Examples

```
lightcurve-analysis --sampler dynesty  # NMMA entry point for kilonova light-curve inference; see tutorials/tutorial-lightcurve_simulation.ipynb for model evaluation via lightcurve-generation
```

## Evaluation signals

- The fitted kilonova model must well describe AT2017gfo: the best-fit light curve with credible band should reproduce the observed multi-band light curves as in Figure 2 of the paper.
- R_1.4 credible interval from the analysis must overlap the joint GW+KN+GRB estimate R = 11.98 (+0.35/−0.40) km.
- GW+KN-only posteriors should be statistically consistent with joint GW+KN+GRB posteriors, with only a 'minimal, yet, noticeable' change in credible-interval widths.
- Posterior samples are stored in the expected NMMA output (posterior_samples.hdf5 / JSON) with named parameters (dynamical and wind ejecta masses, inclination, R_1.4).
- SVD model quality can be checked with the svdmodel-benchmark entry point before inference.

## Limitations

- Limited physics input in kilonova models is an additional, unquantified-by-the-framework uncertainty in the inferred ejecta masses and radius.
- In the joint analysis the improvement in parameter uncertainties from adding GRB170817A to GW+KN is minimal, so kilonova-only inference alone carries weaker EOS constraints.
- NMMA kilonova inference is parameter-estimation infrastructure, not a substitute for improved astrophysical transient models needed for reliable interpretation of future BNS detections.

## Evidence

- [results] both the kilonova AT2017gfo and the GRB afterglow GRB170817A are well described in our analysis: "We find that both the kilonova AT2017gfo and the GRB afterglow GRB170817A are well described in our analysis."
- [methods] Entry points nmma-analysis / lightcurve-analysis: "nmma-analysis` `
- `lightcurve-analysis`"
- [methods] dynesty sampler citation: [evidence span withheld — anchor status: not_found]
- [readme] Lightcurve simulation tutorial notebook: "A tutorial on how to produce simulations of lightcurves is given here [tutorial-lightcurve_simulation.ipynb]"
- [readme] bilby as backend: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
