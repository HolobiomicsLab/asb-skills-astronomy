---
name: bayesian-eos-prior-reweighting
description: Use when a Bayesian EOS-inference pipeline (NMMA/bilby) must incorporate
  radio-timing pulsar mass measurements and NICER mass-radius posteriors as prior
  likelihoods over a precomputed EOS sample, each with an M-R curve and maximum mass
  M_TOV.
when_to_use_negative:
- The GW/kilonova/GRB likelihoods are being computed rather than the EOS prior terms
  — that is the separate NMMA multi-messenger analysis, not the prior reweighting
  step
- Input is raw pulsar timing data or posterior samples that must first be reduced
  to Gaussian mass posteriors or a joint M-R posterior density
edam_topics:
- http://edamontology.org/topic_3388
tools:
- name: NMMA
  role: Multi-messenger inference framework providing the EOS-constraint analysis
    pipeline into which the pulsar/NICER prior likelihoods are incorporated
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend providing the Likelihood interface through which the mass-only pulsar
    likelihood is implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:577314d65717643c819bbc1dba5abfba3d086c451e3d0e149033f8944e50bd68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3388
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bayesian-eos-prior-reweighting

## Summary

Reweights a candidate neutron-star equation-of-state (EOS) sample by evaluating pulsar mass-posterior and NICER mass-radius likelihoods within the NMMA/bilby Bayesian framework, encoding dense-matter information (e.g. the ~2 solar-mass pulsars PSR J0348+0432 and PSR J1614-2230) as prior likelihood terms on each EOS.

## When to use

Use when a Bayesian EOS-inference pipeline (NMMA/bilby) must incorporate radio-timing pulsar mass measurements and NICER mass-radius posteriors as prior likelihoods over a precomputed EOS sample, each with an M-R curve and maximum mass M_TOV.

## When NOT to use

- The GW/kilonova/GRB likelihoods are being computed rather than the EOS prior terms — that is the separate NMMA multi-messenger analysis, not the prior reweighting step
- Input is raw pulsar timing data or posterior samples that must first be reduced to Gaussian mass posteriors or a joint M-R posterior density

## Inputs

- Candidate EOS sample (per-EOS mass-radius curve and maximum mass M_TOV), loadable via numpy/pandas
- Gaussian pulsar mass posteriors for PSR J0348+0432 and PSR J1614-2230
- NICER joint mass-radius posterior for PSR J0740+6620

## Outputs

- eos_prior_loglike.csv containing per-EOS summed pulsar+NICER log-likelihood values

## How to apply

Load the candidate EOS sample with numpy/pandas, where each EOS carries its mass-radius curve and maximum mass M_TOV. Implement the mass-only pulsar likelihood through the bilby Likelihood interface used by NMMA: for PSR J0348+0432 and PSR J1614-2230, evaluate the Gaussian mass posteriors integrated over neutron-star masses up to the EOS's M_TOV using a truncated-Gaussian CDF computed via the error function (erf). For the NICER joint mass-radius likelihood on PSR J0740+6620, evaluate the joint M-R posterior density at the radius the candidate EOS predicts at the pulsar's measured mass. Sum both log-likelihood terms for every EOS and write per-EOS values to eos_prior_loglike.csv; rationale is that ~2 solar-mass pulsars probe densities beyond inspiral gravitational waves and supply a lower bound on the maximum neutron-star mass through the pulsar mass prior likelihood.

## Related tools

- **NMMA** (Multi-messenger inference framework providing the EOS-constraint analysis pipeline into which the pulsar/NICER prior likelihoods are incorporated) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend providing the Likelihood interface through which the mass-only pulsar likelihood is implemented)

## Examples

```
from scipy.special import erf; logL_pulsar = -0.5*((M-mu)/sig)**2 - log(erf((M_TOV-mu)/(sqrt(2)*sig)) - erf((Mlow-mu)/(sqrt(2)*sig)))  # truncated-Gaussian mass likelihood per EOS
```

## Evaluation signals

- All evaluated log-likelihoods must be finite (no NaN/inf in eos_prior_loglike.csv)
- Every EOS whose M_TOV falls below the low-mass tail of the pulsar mass posteriors must receive a heavily suppressed (large negative) log-likelihood
- Resulting EOS constraint should be consistent with the published behaviour, R_1.4 = 11.98 (+0.35/-0.40) km
- Per-EOS log-likelihood must equal the sum of the pulsar mass term and the NICER joint M-R term, so spot-check one EOS by recomputing both components independently

## Limitations

- Additional uncertainties arise from limited physics input in kilonova and semi-analytic GRB models within the broader NMMA analysis
- EOSs with M_TOV below the low-mass tail of the pulsar posteriors cannot be meaningfully recovered — they are only assigned heavily suppressed likelihoods, so the prior cannot distinguish among such EOSs
- The NICER term requires a well-defined joint M-R posterior; an EOS's radius must be evaluable at the pulsar's measured mass

## Evidence

- [other] Radio mass measurements of ~2 solar-mass pulsars provide EOS information at densities beyond those probed by inspiral gravitational waves, and additionally supply a lower bound on the maximum neutron-star mass — the information encoded in the pulsar mass prior likelihood.: [evidence span withheld — anchor status: not_found]
- [abstract] Incorporating all available information, we estimate the radius of a 1.4M⊙ neutron star to be R=11.98−0.40+0.35 km.: [evidence span withheld — anchor status: not_found]
- [discussion] Simultaneous analysis of GW and EM signals with constraints from nuclear-physics calculations and radio/X-ray measurements of isolated NSs: "This framework allows for the simultaneous analysis of GW and EM signals such as kilonovae and GRB afterglows. In addition, our framework allows us to incorporate"
- [readme] NMMA uses bilby as its backend, providing the Likelihood interface for the pulsar mass likelihood.: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
