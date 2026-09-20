---
name: credible-interval-width-quantification
description: Use when posterior samples or published credible intervals must be reduced
  to a single scalar (interval width) to decide whether one analysis genuinely shrinks
  uncertainty relative to a baseline, e.g. R_1.4 = 11.98 (+0.35/−0.40) km vs prior
  literature rows.
when_to_use_negative:
- Inputs are raw event data (GW strain, kilonova photometry, GRB afterglow light curve)
  rather than posterior samples or published intervals — inference must run first
  via nmma-analysis/lightcurve-analysis
- Comparing frequentist confidence intervals or error bars from non-Bayesian fits,
  where credible-interval width is not the appropriate uncertainty summary
- Only a single posterior exists and there is no baseline (literature or alternate
  configuration) to compare widths against
edam_topics:
- http://edamontology.org/topic_0268
tools:
- name: NMMA
  role: Framework whose joint (GW+KN+GRB) and GW+KN-only analyses produce the posteriors/credible
    intervals being width-quantified and compared
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Inference backend used by NMMA to run the Bayesian parameter estimation whose
    posterior intervals are measured
- name: dynesty
  role: Nested sampler (run via bilby) generating posterior samples for R_1.4 and
    kilonova ejecta parameters
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:4de23d2289ebe6a8bd1ee9e82a0e31988325fb663bf4357784da912da6a0f68b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  - dynesty
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0268
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# credible-interval-width-quantification

## Summary

Quantifies Bayesian credible-interval widths (upper minus lower bound) for key parameters such as the 1.4 solar-mass neutron-star radius R_1.4, and compares them across analysis configurations (GW+KN only vs joint GW+KN+GRB) and against literature baselines (Dietrich 2020, Essick 2020, Breschi 2022, Nicholl 2022, Raaijmakers 2021, Miller 2021, Huth 2022) to verify claimed uncertainty improvements.

## When to use

Use when posterior samples or published credible intervals must be reduced to a single scalar (interval width) to decide whether one analysis genuinely shrinks uncertainty relative to a baseline, e.g. R_1.4 = 11.98 (+0.35/−0.40) km vs prior literature rows.

## When NOT to use

- Inputs are raw event data (GW strain, kilonova photometry, GRB afterglow light curve) rather than posterior samples or published intervals — inference must run first via nmma-analysis/lightcurve-analysis
- Comparing frequentist confidence intervals or error bars from non-Bayesian fits, where credible-interval width is not the appropriate uncertainty summary
- Only a single posterior exists and there is no baseline (literature or alternate configuration) to compare widths against

## Inputs

- Published credible intervals (central value + lower/upper bounds) per reference, e.g. the eight Table 1 entries of R_1.4 estimates
- Posterior sample sets from NMMA nested sampling, e.g. gw_kn_only_posterior_samples.json and joint GW+KN+GRB posterior samples
- NMMA joint-analysis estimate R_1.4 = 11.98 (+0.35/−0.40) km

## Outputs

- radius_comparison_table.csv with columns reference, radius central value, lower bound, upper bound, and credible-interval width
- posterior_comparison_gw_kn_vs_joint.png corner-style figure with overlaid R_1.4 and ejecta-parameter marginals
- Quantified change in credible-interval widths between analysis configurations

## How to apply

Extract, for each entry (e.g. the eight Table 1 rows of radius estimates), the central value and lower/upper credible-interval bounds, and assemble them into a pandas DataFrame (reference, radius central value, lower bound, upper bound) written to radius_comparison_table.csv. Compute the credible-interval width as upper minus lower bound per row and add it as a column; for the paper's joint NMMA analysis the width is 0.35 + 0.40 = 0.75 km. Then compare the 'this work' width against the seven prior-publication widths and confirm it is noticeably narrower, consistent with the claim that adding GRB170817A to GW170817 + AT2017gfo gives a 'minimal, yet, noticeable' improvement. For self-computed posteriors, run nested sampling (dynesty via bilby as the NMMA backend) on the GW+KN-only likelihood, save posterior samples (e.g. gw_kn_only_posterior_samples.json), overlay R_1.4 and ejecta-mass marginals against the joint GW+KN+GRB posteriors in a corner-style figure (posterior_comparison_gw_kn_vs_joint.png), and quantify the change in credible-interval widths. Rationale: posterior-consistency plus a visibly reduced interval width is the paper's operational definition of an improvement from multi-messenger joint inference.

## Related tools

- **NMMA** (Framework whose joint (GW+KN+GRB) and GW+KN-only analyses produce the posteriors/credible intervals being width-quantified and compared) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Inference backend used by NMMA to run the Bayesian parameter estimation whose posterior intervals are measured)
- **dynesty** (Nested sampler (run via bilby) generating posterior samples for R_1.4 and kilonova ejecta parameters)

## Examples

```
import pandas as pd; df=pd.read_csv('radius_comparison_table.csv'); df['ci_width']=df['upper_bound']-df['lower_bound']; print(df.sort_values('ci_width'))
```

## Evaluation signals

- The 'this work' row reproduces R = 11.98 km with +0.35/−0.40 km bounds, i.e. total credible-interval width ≈ 0.75 km
- The computed width column (upper − lower) is present and positive for all eight reference rows
- The 'this work' credible-interval width is visibly narrower than the majority of the seven prior-literature widths
- The GW+KN-only R_1.4 credible interval overlaps the joint R_1.4 = 11.98 (+0.35/−0.40) km estimate (posterior consistency)
- Uncertainty improvement from adding GRB170817A is 'minimal, yet, noticeable' — width reduction is small but non-zero, not a dramatic shrinkage

## Limitations

- Improvement from adding GRB170817A is only minimal, yet noticeable — do not over-interpret small width reductions as substantial gains
- Kilonova-model and semi-analytic GRB-model physics limitations add unquantified systematic uncertainty on top of the reported intervals
- No significant EOS constraint is imposed by jet energy E0 because the ratio ξ between E0 and the disk mass mdisk is a free parameter; width improvements partly reflect inclination constraints routed through ejecta masses

## Evidence

- [abstract] Joint NMMA R_1.4 estimate and interval to reproduce: [evidence span withheld — anchor status: not_found]
- [results] Expected magnitude of improvement when adding GRB data: "the improvement on the parameter uncertainties is minimal, yet, noticeable when information from GRB170817A is added."
- [methods] Inference backend and sampling stack used to produce the posteriors: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
- [intro] Need for statistical improvement criteria in joint analysis: "results have been obtained by connecting constraints from individual messengers a posteriori, i.e., different messengers were analyzed individually and then combined within different multi-messenger"
