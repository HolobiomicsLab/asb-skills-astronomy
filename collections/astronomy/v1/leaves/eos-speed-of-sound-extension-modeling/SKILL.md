---
name: eos-speed-of-sound-extension-modeling
description: Use when you must extend a nuclear-physics-constrained EOS (chiral-EFT/AFDMC
  samples, valid to ~1.5 nsat) to supra-nuclear densities to obtain mass-radius-tidal-deformability
  families for neutron-star inference.
when_to_use_negative:
- The input EOS is already valid across the full density range you need (no supra-nuclear
  extension required, e.g., EOS bands already tabulated beyond the chiral-EFT limit).
- The analysis target is kilonova/GRB afterglow light-curve modeling (nmma-generation,
  lightcurve-analysis entry points) rather than EOS construction.
- You need EOS constraints only below ~1.5 nsat, where the chiral-EFT band itself
  suffices without a speed-of-sound model.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3450
- http://edamontology.org/topic_0203
tools:
- name: NMMA
  role: Framework hosting the 'combine-EOS' entry point used to load low-density EOS
    samples and construct the speed-of-sound extension
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend providing prior distributions for speed-of-sound parameters and likelihood
    terms (e.g., pQCD constraint)
  repo: https://arxiv.org/pdf/1811.02042
- name: dynesty
  role: Nested sampler used via the bilby backend to obtain the EOS posterior
  repo: https://zenodo.org/records/17268284
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:d6262a74df1097ed875405e2de73e56320e9b2443161a4f5233b39c9c4ca1d29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  - dynesty
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3450
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# eos-speed-of-sound-extension-modeling

## Summary

Constructs candidate neutron-star equations of state by combining a low-density band constrained by Quantum Monte Carlo calculations with chiral EFT interactions (up to 1.5 nsat) with a parametric speed-of-sound extension at higher densities, within the NMMA framework.

## When to use

Use when you must extend a nuclear-physics-constrained EOS (chiral-EFT/AFDMC samples, valid to ~1.5 nsat) to supra-nuclear densities to obtain mass-radius-tidal-deformability families for neutron-star inference.

## When NOT to use

- The input EOS is already valid across the full density range you need (no supra-nuclear extension required, e.g., EOS bands already tabulated beyond the chiral-EFT limit).
- The analysis target is kilonova/GRB afterglow light-curve modeling (nmma-generation, lightcurve-analysis entry points) rather than EOS construction.
- You need EOS constraints only below ~1.5 nsat, where the chiral-EFT band itself suffices without a speed-of-sound model.

## Inputs

- Low-density nuclear EOS uncertainty band samples (AFDMC / chiral-EFT interactions), constrained up to 1.5 nsat
- Speed-of-sound prior definitions in bilby (1-parameter soft/stiff interpolation; 5 linear segments, max density 12*nsat)
- Optional pQCD thermodynamic band at ~40-50 nsat for asymptotic-density constraints
- Per-constraint EOS sample files from upstream NMMA analyses

## Outputs

- EOS families as mass-radius-Lambda arrays (e.g., eos_samples_pqcd_constrained.dat)
- TOV-derived mass-radius sequences per sampled EOS
- Causal, thermodynamically stable EOS posterior samples with derived neutron-star properties (e.g., R_1.4)

## How to apply

Load the low-density nuclear EOS uncertainty band (AFDMC / chiral-EFT samples) through the NMMA 'combine-EOS' entry point and sample the 1-parameter soft/stiff interpolation across the band to draw candidate EOS realizations. For densities above 2*nsat, construct the high-density extension with the parametric speed-of-sound scheme using the default settings of a maximum density of 12*nsat and 5 linear segments in the sound-speed parameterization, drawing speed-of-sound values through priors defined in bilby. For each sampled EOS, solve the TOV equations to obtain the mass-radius sequence and compute the tidal deformability Lambda along each mass-radius family from the TOV solution. Additional physical inputs (e.g., a pQCD thermodynamic band at ~40-50 nsat as a likelihood term in bilby) can be layered on, followed by nested sampling with dynesty to obtain the constrained EOS posterior. Save each EOS family as mass-radius-Lambda arrays (e.g., eos_samples_pqcd_constrained.dat) after screening for causality and thermodynamic stability.

## Related tools

- **NMMA** (Framework hosting the 'combine-EOS' entry point used to load low-density EOS samples and construct the speed-of-sound extension) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend providing prior distributions for speed-of-sound parameters and likelihood terms (e.g., pQCD constraint)) — https://arxiv.org/pdf/1811.02042
- **dynesty** (Nested sampler used via the bilby backend to obtain the EOS posterior) — https://zenodo.org/records/17268284

## Examples

```
nmma combine-EOS --eos-samples low_density_chiral_eft.dat --extension speed-of-sound --max-density 12*nsat --segments 5 --sampler dynesty
```

## Evaluation signals

- Every generated EOS family reaches a maximum mass consistent with the known ~2 solar-mass pulsar constraints.
- Every retained EOS satisfies causality (speed of sound does not exceed the speed of light) and thermodynamic stability (monotonically increasing pressure with energy density) at all densities up to the pQCD matching density.
- The radius of a 1.4 solar-mass neutron star from the posterior remains consistent with the published NMMA estimate of R_1.4 ≈ 11.98 km (+0.35/-0.40 km).
- Low-density portion of each EOS lies within the chiral-EFT/AFDMC uncertainty band up to 1.5 nsat.
- Output files contain mass-radius-Lambda arrays for every sampled EOS family.

## Limitations

- The EOS families are constrained only up to 1.5 nsat by Quantum Monte Carlo with chiral EFT interactions; the asymptotically high-density regime remains unconstrained unless an additional input such as perturbative QCD is supplied.
- Default speed-of-sound parameterization choices (maximum density 12*nsat, 5 linear segments) are model assumptions that shape the high-density extension.
- NMMA's kilonova and semi-analytic GRB models have limited physics input, which propagates as additional uncertainty in multi-messenger analyses using these EOS families.
- No version number or DOI for the publicly available NMMA framework is provided in the discussion text; users must cite the package by name and the companion paper.

## Evidence

- [results] Low-density constraint and extension structure: [evidence span withheld — anchor status: not_found]
- [results] Default parameterization settings: [evidence span withheld — anchor status: not_found]
- [abstract] R_1.4 benchmark: [evidence span withheld — anchor status: not_found]
- [results] pQCD extension at asymptotic densities: [evidence span withheld — anchor status: not_found]
- [readme] bilby backend citation requirement: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
