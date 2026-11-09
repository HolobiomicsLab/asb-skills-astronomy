---
name: tidal-deformability-computation
description: Use when a candidate EOS realization (chiral-EFT low-density band extended
  by a speed-of-sound model) must be converted into a mass-radius and tidal-deformability
  family for binary-neutron-star inference.
when_to_use_negative:
- The signal is a binary black hole merger or other system with no material/tidal
  effects, so Lambda is undefined
- Only kilonova light-curve or GRB afterglow fitting is needed without any EOS-to-mass-radius
  conversion (use nmma lightcurve-analysis entry points instead)
- The input is already a tabulated mass-radius-Lambda family, e.g. from a prior TOV
  solve — re-solving would duplicate the step
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: NMMA
  role: Hosts the 'combine-EOS' entry point and EOS utilities used to sample the chiral-EFT
    low-density band, build the speed-of-sound extension, solve the TOV equations,
    and compute Lambda per family
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend through which NMMA runs the downstream Bayesian inference that consumes
    the mass-radius-Lambda arrays
- name: dynesty
  role: Nested sampler used for the analysis of the EOS/inference runs built on the
    computed Lambda families
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:14b6b5099fa7e9501fca07baeadb7ffe35d66b1908ad3ae92a6b8308ed5546fd
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
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tidal-deformability-computation

## Summary

Computes the tidal deformability (Lambda) of neutron stars along a sampled equation-of-state (EOS) family by solving the TOV equations within the NMMA framework, producing mass-radius-Lambda arrays used as BNS gravitational-wave matter constraints.

## When to use

Use when a candidate EOS realization (chiral-EFT low-density band extended by a speed-of-sound model) must be converted into a mass-radius and tidal-deformability family for binary-neutron-star inference.

## When NOT to use

- The signal is a binary black hole merger or other system with no material/tidal effects, so Lambda is undefined
- Only kilonova light-curve or GRB afterglow fitting is needed without any EOS-to-mass-radius conversion (use nmma lightcurve-analysis entry points instead)
- The input is already a tabulated mass-radius-Lambda family, e.g. from a prior TOV solve — re-solving would duplicate the step

## Inputs

- Low-density nuclear EOS uncertainty band samples (AFDMC / chiral-EFT, constrained up to 1.5 nsat)
- Soft/stiff 1-parameter interpolation samples across the low-density band
- Speed-of-sound extension parameters (max density 12*nsat, 5 linear segments above 2*nsat)
- Known ~2 solar-mass pulsar mass constraints for filtering

## Outputs

- Mass-radius sequence per sampled EOS from the TOV solution
- Mass-radius-Lambda arrays per EOS family
- Filtered set of EOS families passing the ~2 solar-mass maximum-mass check

## How to apply

Load the low-density nuclear EOS uncertainty band (AFDMC / chiral-EFT samples) into the NMMA EOS utilities exposed through the 'combine-EOS' entry point, sampling the 1-parameter soft/stiff interpolation across the low-density band, which is constrained up to 1.5 nsat by Quantum Monte Carlo calculations. For densities above 2*nsat, build the high-density extension with the parametric speed-of-sound scheme using the defaults of a maximum density of 12*nsat and 5 linear segments in the sound-speed parameterization. Solve the TOV equations for each sampled EOS to obtain the mass-radius sequence, then compute the tidal deformability Lambda along each mass-radius family from the TOV solution. Save each family as mass-radius-Lambda arrays and verify that every generated family reaches a maximum mass consistent with the known ~2 solar-mass pulsar constraints before feeding the arrays into the NMMA/Bilby inference (e.g., dynesty or PyMultiNest samplers).

## Related tools

- **NMMA** (Hosts the 'combine-EOS' entry point and EOS utilities used to sample the chiral-EFT low-density band, build the speed-of-sound extension, solve the TOV equations, and compute Lambda per family) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend through which NMMA runs the downstream Bayesian inference that consumes the mass-radius-Lambda arrays)
- **dynesty** (Nested sampler used for the analysis of the EOS/inference runs built on the computed Lambda families)

## Evaluation signals

- Every generated mass-radius-Lambda family reaches a maximum mass consistent with the known ~2 solar-mass pulsar constraints
- Lambda values are computed from the TOV solution (not read directly from the EOS), and each family is saved as mass-radius-Lambda arrays
- The high-density speed-of-sound extension joins the chiral-EFT low-density part at the 1.5-2 nsat transition using the default 12*nsat maximum density and 5 linear segments
- Downstream multi-messenger inference with the computed families reproduces article-scale results, e.g. R_1.4 = 11.98 (+0.35/-0.40) km when all constraints are incorporated
- The 'combine-EOS' entry point from the NMMA methods section is the entry point actually invoked for the EOS-to-family conversion

## Limitations

- The speed-of-sound extension is a parametric approximation (5 linear segments up to 12*nsat), so high-density EOS behavior depends on this parameterization choice
- Limited physics input in kilonova and semi-analytic GRB models adds additional uncertainty to multi-messenger constraints that rely on these EOS families
- No significant EOS constraint arises from the GRB jet energy E0 because the ratio xi between E0 and the disk mass mdisk is a free parameter; the EOS constraint enters mainly via inclination-driven kilonova ejecta masses

## Evidence

- [other] TOV-and-Lambda pipeline: [evidence span withheld — anchor status: not_found]
- [other] Speed-of-sound extension defaults: [evidence span withheld — anchor status: not_found]
- [abstract] EOS-constraining purpose of NMMA: "the NMMA code has allowed us to constrain the equation of state of supranuclear dense matter, to measure the Hubble constant"
- [discussion] Simultaneous GW+EM framework: "This framework allows for the simultaneous analysis of GW and EM signals such as kilonovae and GRB afterglows."
- [readme] README scope statement: "a pythonic library for probing nuclear physics and cosmology with multimessenger analysis"
