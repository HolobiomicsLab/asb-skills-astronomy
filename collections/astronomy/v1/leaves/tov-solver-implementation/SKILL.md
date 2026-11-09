---
name: tov-solver-implementation
description: Use when candidate neutron-star EOSs sampled from a low-density chiral-EFT/AFDMC
  band must be extended to high density and converted into mass-radius-Lambda arrays
  via the TOV equations for astrophysical (e.g., GW170817-style) constraints.
when_to_use_negative:
- The task is kilonova or GRB-afterglow light-curve modeling via NMMA's lightcurve-analysis
  / create-lightcurve-slurm entry points rather than EOS structure and TOV solving.
- The inputs are already mass-radius-Lambda arrays from a prior TOV run and only need
  likelihood evaluation or posterior combination, not a new solve.
- The density regime of interest lies entirely below 1.5 nsat, where the chiral-EFT
  Quantum Monte Carlo band is sufficient and no speed-of-sound extension is needed.
tools:
- name: NMMA
  role: Hosts the combine-EOS entry point and EOS utilities used to stitch the chiral-EFT
    low-density band to the speed-of-sound extension and run the TOV solver for each
    sampled EOS.
  repo: github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend of NMMA's Bayesian inference layer that consumes the TOV-derived mass-radius-Lambda
    arrays in the EOS likelihood.
- name: dynesty
  role: Nested sampler used (with bilby) to explore the EOS parameter space whose
    realizations are evaluated through the TOV solver.
  repo: https://zenodo.org/records/17268284
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:2521f1fee813510c44070b3d2c2d4a79aaeac1eaa9f8076b55d523d3a37fcd32
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NMMA
  - bilby
  - dynesty
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# TOV solver implementation for neutron-star EOS structure

## Summary

Implements the Tolman–Oppenheimer–Volkoff (TOV) solve inside the NMMA framework to convert candidate EOSs — chiral-EFT-constrained up to 1.5 nsat and extended above 2*nsat with a parametric speed-of-sound model — into mass-radius sequences and tidal deformability (Lambda) predictions, enabling EOS candidates to be tested against ~2 solar-mass pulsar constraints.

## When to use

Use when candidate neutron-star EOSs sampled from a low-density chiral-EFT/AFDMC band must be extended to high density and converted into mass-radius-Lambda arrays via the TOV equations for astrophysical (e.g., GW170817-style) constraints.

## When NOT to use

- The task is kilonova or GRB-afterglow light-curve modeling via NMMA's lightcurve-analysis / create-lightcurve-slurm entry points rather than EOS structure and TOV solving.
- The inputs are already mass-radius-Lambda arrays from a prior TOV run and only need likelihood evaluation or posterior combination, not a new solve.
- The density regime of interest lies entirely below 1.5 nsat, where the chiral-EFT Quantum Monte Carlo band is sufficient and no speed-of-sound extension is needed.

## Inputs

- Low-density chiral-EFT / AFDMC EOS uncertainty-band samples (valid up to 1.5 nsat)
- Sampled candidate EOS realizations (1-parameter soft/stiff interpolation across the low-density band)
- Speed-of-sound extension settings (default maximum density 12*nsat, 5 linear segments in the sound-speed parameterization)

## Outputs

- Mass-radius sequence per sampled EOS (from the TOV solution)
- Tidal deformability Lambda along each mass-radius family
- Saved mass-radius-Lambda arrays per EOS family, filtered for consistency with the ~2 solar-mass pulsar maximum-mass constraint

## How to apply

Load the low-density nuclear EOS uncertainty band (AFDMC / chiral-EFT samples) into the NMMA EOS utilities exposed through the 'combine-EOS' entry point, and sample the 1-parameter soft/stiff interpolation across the low-density band (valid up to 1.5 nsat, where Quantum Monte Carlo with chiral EFT interactions constrains the EOS). For densities above 2*nsat, build the high-density extension with the parametric speed-of-sound scheme using NMMA's defaults: maximum density 12*nsat and 5 linear segments in the sound-speed parameterization. For each sampled EOS realization, solve the TOV equations to obtain the mass-radius sequence, then compute the tidal deformability Lambda along each mass-radius family from the TOV solution. Save each family as mass-radius-Lambda arrays and reject any family whose maximum mass is inconsistent with the known ~2 solar-mass pulsar constraints. The resulting arrays feed the NMMA/bilby likelihood (nested sampling via dynesty) that combines gravitational-wave, kilonova, and GRB-afterglow information with nuclear-physics EOS priors.

## Related tools

- **NMMA** (Hosts the combine-EOS entry point and EOS utilities used to stitch the chiral-EFT low-density band to the speed-of-sound extension and run the TOV solver for each sampled EOS.) — github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend of NMMA's Bayesian inference layer that consumes the TOV-derived mass-radius-Lambda arrays in the EOS likelihood.)
- **dynesty** (Nested sampler used (with bilby) to explore the EOS parameter space whose realizations are evaluated through the TOV solver.) — https://zenodo.org/records/17268284

## Examples

```
combine-EOS
```

## Evaluation signals

- Every generated mass-radius family reaches a maximum mass consistent with the known ~2 solar-mass pulsar constraints; families that fall short must be flagged/rejected.
- The speed-of-sound extension used the NMMA default settings: maximum density 12*nsat and 5 linear segments in the sound-speed parameterization (verify these parameters in the run configuration).
- The low-density portion of each candidate EOS lies within the chiral-EFT/AFDMC uncertainty band up to 1.5 nsat.
- Each EOS family is saved as a complete mass-radius-Lambda array, with Lambda computed consistently from the same TOV solution as the radius sequence.
- Resulting R_1.4 (radius of a 1.4 solar-mass neutron star) values fall in the plausible range around the published estimate R = 11.98 (+0.35/-0.40) km when the full multi-messenger analysis is applied.

## Limitations

- The high-density behavior is phenomenological: the speed-of-sound extension with defaults of 12*nsat and 5 linear segments is a parameterized model, not ab initio nuclear physics, so results depend on these parameterization choices.
- EOS families that never reach the ~2 solar-mass maximum mass are unusable, so extension parameters must be validated per sample before the family enters the inference.
- Downstream multi-messenger interpretation carries additional uncertainties from limited physics input in the kilonova and semi-analytic GRB models, which limits what the TOV-derived EOS constraints can be expected to resolve.

## Evidence

- [other] Solve the TOV equations for each sampled EOS to obtain the mass-radius sequence.: [evidence span withheld — anchor status: not_found]
- [other] Low-density chiral-EFT constraint extent: [evidence span withheld — anchor status: not_found]
- [abstract] EOS constraining power of NMMA: "the NMMA code has allowed us to constrain the equation of state of supranuclear dense matter, to measure the Hubble constant"
- [discussion] Multi-messenger and nuclear-physics combination: "This framework allows for the simultaneous analysis of GW and EM signals such as kilonovae and GRB afterglows. In addition, our framework allows us to incorporate constraints from nuclear-physics"
- [methods] bilby as NMMA backend: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
- [readme] NMMA library purpose: "a pythonic library for probing nuclear physics and cosmology with multimessenger analysis"
