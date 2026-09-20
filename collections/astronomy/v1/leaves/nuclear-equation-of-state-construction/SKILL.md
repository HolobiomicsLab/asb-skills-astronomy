---
name: nuclear-equation-of-state-construction
description: 'Use when building candidate EOS realizations for binary-neutron-star
  analyses: you have a low-density nuclear EOS uncertainty band (chiral EFT / AFDMC
  samples) and need high-density extensions, TOV mass-radius sequences, and Lambda
  values for sampling.'
when_to_use_negative:
- You are analyzing kilonova or GRB afterglow light curves with the 'lightcurve-analysis'
  / 'nmma-analysis' entry points rather than generating EOS samples
- Your analysis only needs pre-existing EOS tables and does not require sampling a
  high-density speed-of-sound extension beyond the chiral-EFT band
- The regime of interest is below 1.5 nsat, where the chiral-EFT/QMC band already
  constrains the EOS directly and no extension is needed
tools:
- name: NMMA
  role: Framework exposing the 'combine-EOS' entry point and EOS utilities used to
    sample the chiral-EFT low-density band, build the speed-of-sound extension, and
    solve the TOV equations
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend used by NMMA for the Bayesian inference underlying the EOS sampling
    analysis
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: dynesty
  role: Nested sampler used for the analysis when running the NMMA EOS workflut through
    its backend
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:fd581c0e7050ce747cec2e9c8fd47ab9679cd57ac279e54b38c975eac2824eef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  - dynesty
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# nuclear-equation-of-state-construction

## Summary

Constructs candidate neutron-star equations of state by combining chiral-EFT/Quantum Monte Carlo constraints on low-density matter (up to ~1.5 nsat) with a parametric speed-of-sound extension at high density, then solving the TOV equations to obtain mass-radius-tidal-deformability families inside the NMMA framework.

## When to use

Use when building candidate EOS realizations for binary-neutron-star analyses: you have a low-density nuclear EOS uncertainty band (chiral EFT / AFDMC samples) and need high-density extensions, TOV mass-radius sequences, and Lambda values for sampling.

## When NOT to use

- You are analyzing kilonova or GRB afterglow light curves with the 'lightcurve-analysis' / 'nmma-analysis' entry points rather than generating EOS samples
- Your analysis only needs pre-existing EOS tables and does not require sampling a high-density speed-of-sound extension beyond the chiral-EFT band
- The regime of interest is below 1.5 nsat, where the chiral-EFT/QMC band already constrains the EOS directly and no extension is needed

## Inputs

- Low-density nuclear EOS uncertainty band (AFDMC / chiral-EFT samples) valid up to ~1.5 nsat
- Speed-of-sound extension settings (maximum density 12*nsat, 5 linear segments in the sound-speed parameterization)
- Known pulsar mass constraints (~2 solar-mass) used as a validity check

## Outputs

- Sampled EOS realizations spanning the low-density band plus speed-of-sound extension
- TOV mass-radius sequences per sampled EOS
- Tidal deformability (Lambda) along each mass-radius family
- EOS families saved as mass-radius-Lambda arrays

## How to apply

Load the low-density nuclear EOS uncertainty band (AFDMC / chiral-EFT samples) through the NMMA 'combine-EOS' entry point, and sample a 1-parameter soft/stiff interpolation across that band to draw candidate EOS realizations. For densities above 2*nsat, construct the high-density extension with the parametric speed-of-sound scheme using the default settings of a maximum density of 12*nsat and 5 linear segments in the sound-speed parameterization. For each sampled EOS, solve the TOV equations to obtain the mass-radius sequence and compute the tidal deformability Lambda along each family. Save each EOS family as mass-radius-Lambda arrays and verify that every generated family reaches a maximum mass consistent with the known ~2 solar-mass pulsar constraints. The framework then permits incorporating these EOS sets together with nuclear-physics and multi-messenger (GW/kilonova/GRB) constraints in a joint analysis.

## Related tools

- **NMMA** (Framework exposing the 'combine-EOS' entry point and EOS utilities used to sample the chiral-EFT low-density band, build the speed-of-sound extension, and solve the TOV equations) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend used by NMMA for the Bayesian inference underlying the EOS sampling analysis) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **dynesty** (Nested sampler used for the analysis when running the NMMA EOS workflut through its backend) — https://github.com/nuclear-multimessenger-astronomy/nmma

## Examples

```
combine-EOS
```

## Evaluation signals

- Every generated EOS family reaches a maximum mass consistent with the known ~2 solar-mass pulsar constraints
- Each sampled EOS yields a converged TOV mass-radius sequence with an associated Lambda computed along the family
- Output arrays follow the mass-radius-Lambda schema for every EOS family
- The low-density portion of each realization lies within the chiral-EFT/AFDMC uncertainty band up to ~1.5 nsat
- The high-density extension above 2*nsat uses the speed-of-sound parameterization with 5 linear segments up to 12*nsat

## Limitations

- The extension is a parametric speed-of-sound model; physical realism above 2*nsat depends on the choice of segment parameterization
- EOS families failing the ~2 solar-mass maximum-mass check must be rejected, reducing sampling efficiency
- The NMMA framework also carries separate uncertainties from limited physics input in kilonova and semi-analytic GRB models, which are distinct from EOS construction errors

## Evidence

- [methods] combine-EOS entry point listed among NMMA CLI entry points: "- `combine-EOS`
- `create-lightcurve-slurm`"
- [discussion] NMMA incorporates nuclear-physics EOS constraints: "our framework allows us to incorporate constraints from nuclear-physics"
- [intro] NMMA previously applied to constrain the neutron-star EOS: "our previous Nuclear physics - Multi-Messenger Astrophysics (NMMA) framework has been successfully applied to provide constraints on the EOS of NS matter"
- [abstract] NMMA development as a public framework: "For this purpose, we have developed the Nuclear-physics and Multi-Messenger Astrophysics framework NMMA."
- [readme] README: NMMA repository and bilby backend: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
