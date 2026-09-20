---
name: multi-messenger-inference-uncertainty-comparison
description: Use when a paper reports a parameter estimate (e.g., R_1.4) with asymmetric
  credible bounds from a joint analysis and you must verify a claimed precision improvement
  against Table 1-style literature baselines.
when_to_use_negative:
- The parameter of interest has symmetric errors or no reported credible bounds, so
  upper-minus-lower width cannot be defined consistently across entries
- Baselines come from fundamentally different analyses (e.g., isolated NS X-ray measurements
  vs merger inference) whose credible intervals are not defined on comparable posterior
  assumptions
- The goal is point-estimate accuracy rather than precision comparison — width comparison
  says nothing about which central value is correct
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: NMMA
  role: Joint Bayesian inference framework whose simultaneous GW + kilonova + GRB-afterglow
    analysis produces the 'this work' R_1.4 credible interval being benchmarked
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: pandas
  role: Used to assemble the radius comparison DataFrame and compute credible-interval
    width per reference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:e031bb5e3afee7dce4f49ec8195c6389b5c6cecbbcebaec995db830c7c4943b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# multi-messenger-inference-uncertainty-comparison

## Summary

A tabular comparison procedure for quantifying how much a joint multi-messenger Bayesian analysis (NMMA on GW170817 + AT2017gfo + GRB170817A) narrows the credible interval of a derived parameter — here the 1.4 solar-mass neutron-star radius R = 11.98 (+0.35/−0.40) km — relative to prior single- or partial-messenger literature baselines.

## When to use

Use when a paper reports a parameter estimate (e.g., R_1.4) with asymmetric credible bounds from a joint analysis and you must verify a claimed precision improvement against Table 1-style literature baselines.

## When NOT to use

- The parameter of interest has symmetric errors or no reported credible bounds, so upper-minus-lower width cannot be defined consistently across entries
- Baselines come from fundamentally different analyses (e.g., isolated NS X-ray measurements vs merger inference) whose credible intervals are not defined on comparable posterior assumptions
- The goal is point-estimate accuracy rather than precision comparison — width comparison says nothing about which central value is correct

## Inputs

- Comparison table (e.g., Table 1) of 1.4M⊙ NS radius estimates with central values and lower/upper credible-interval bounds from eight references
- Joint NMMA posterior result for R_1.4 incorporating GW170817, AT2017gfo, and GRB170817A
- Published literature baselines (Dietrich 2020, Essick 2020, Breschi 2022, Nicholl 2022, Raaijmakers 2021, Miller 2021, Huth 2022)

## Outputs

- radius_comparison_table.csv with columns: reference, radius central value, lower credible bound, upper credible bound
- Credible-interval width column (upper minus lower bound) per reference
- Validation verdict: whether 'this work' width (~0.75 km) is noticeably narrower than the majority of prior literature entries

## How to apply

Extract, for each reference entry in the comparison table (Dietrich 2020, Essick 2020, Breschi 2022, Nicholl 2022, Raaijmakers 2021, Miller 2021, Huth 2022, and 'this work'), the central value and lower/upper credible-interval bounds of the 1.4 solar-mass neutron-star radius. Assemble these into a pandas DataFrame with columns for reference, central radius, lower bound, upper bound, and computed credible-interval width (upper minus lower). Locate the 'this work' row produced by the NMMA joint analysis and validate it reproduces R = 11.98 km with +0.35/−0.40 km bounds (total width ≈ 0.75 km). The decision point is the width comparison: the joint NMMA entry must be visibly narrower than the majority of the seven prior entries, confirming that incorporating the GRB170817A afterglow (analyzed with a synthetic Gaussian jet-model lightcurve) on top of the GW and kilonova data yields a 'minimal, yet, noticeable' uncertainty improvement.

## Related tools

- **NMMA** (Joint Bayesian inference framework whose simultaneous GW + kilonova + GRB-afterglow analysis produces the 'this work' R_1.4 credible interval being benchmarked) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **pandas** (Used to assemble the radius comparison DataFrame and compute credible-interval width per reference)

## Examples

```
import pandas as pd; df = pd.read_csv('radius_comparison_table.csv'); df['ci_width'] = df['upper_bound'] - df['lower_bound']; print(df[df['reference']=='this work']['ci_width'])
```

## Evaluation signals

- The 'this work' row reproduces R = 11.98 km with asymmetric bounds +0.35/−0.40 km, giving a total width of ≈0.75 km
- The computed width column equals upper_bound − lower_bound for every one of the eight entries
- The 'this work' credible-interval width is visibly narrower than the widths of the majority of the seven prior publications, consistent with the paper's improvement claim
- The DataFrame contains exactly eight reference entries matching the Table 1 enumeration, written to radius_comparison_table.csv without missing bounds

## Limitations

- The improvement from adding GRB170817A is explicitly 'minimal, yet, noticeable' — the width comparison may not support claims of dramatic precision gains
- Limited physics input in the kilonova and semi-analytic GRB models adds uncertainties to the joint result being benchmarked
- Asymmetric credible bounds (+0.35/−0.40) mean a single width number loses the asymmetry information; both bounds must be retained in the table

## Evidence

- [abstract] Joint NMMA radius result: [evidence span withheld — anchor status: not_found]
- [results] Improvement claim from GRB inclusion: "the improvement on the parameter uncertainties is minimal, yet, noticeable when information from GRB170817A is added."
- [results] GRB afterglow modeling step: "The GRB afterglow light-curve data are analyzed with the synthetic Gaussian jet-model lightcurve described before57,58"
- [discussion] Model-limitation caveat: "additional uncertainties in our work lie in limited physics input in kilonova and semi-analytic GRB and models"
- [readme] Tool citation requirement: "When using this code for a publication, kindly make a reference to the package by its name, NMMA, and a citation to the companion paper"
