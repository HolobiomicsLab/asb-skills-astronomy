---
name: truncated-gaussian-cdf-evaluation
description: Use when scoring a candidate EOS sample against radio-timing pulsar mass
  posteriors (Gaussian in mass, with EOS-dependent support up to M_TOV), e.g. to build
  per-EOS prior log-likelihoods for NMMA neutron-star EOS inference.
when_to_use_negative:
- The quantity of interest is a joint mass-radius likelihood (e.g. NICER for PSR J0740+6620)
  rather than a mass-only likelihood — the NICER term is evaluated from the joint
  M-R posterior, not a truncated Gaussian in mass alone.
- The mass distribution has no EOS-dependent upper bound, so there is no truncation
  at M_TOV and an ordinary (untruncated) Gaussian likelihood suffices.
- You are sampling GW/kilonova event parameters rather than computing per-EOS prior
  weights in the NMMA EOS-constraint workflow.
edam_operation: http://edamontology.org/operation_3659
tools:
- name: NMMA
  role: Framework within which the per-EOS pulsar prior likelihood is implemented
    and combined with GW/kilonova/EM analyses to constrain the neutron-star EOS
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: bilby
  role: Backend whose Likelihood interface is used to implement the mass-only pulsar
    likelihood
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:3382afea0dc8b5374ee488550f8dc6c451dca0c4dd54a0b831d4dd7b2953abd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# truncated-gaussian-cdf-evaluation

## Summary

Evaluates the mass-only pulsar likelihood used by the NMMA EOS-inference framework by integrating Gaussian pulsar mass posteriors over neutron-star masses up to each candidate EOS's maximum mass M_TOV, i.e. a truncated-Gaussian CDF computed via the error function. This encodes how ~2 solar-mass radio-timing pulsars (PSR J0348+0432, PSR J1614-2230) constrain the neutron-star equation of state beyond densities probed by inspiral gravitational waves.

## When to use

Use when scoring a candidate EOS sample against radio-timing pulsar mass posteriors (Gaussian in mass, with EOS-dependent support up to M_TOV), e.g. to build per-EOS prior log-likelihoods for NMMA neutron-star EOS inference.

## When NOT to use

- The quantity of interest is a joint mass-radius likelihood (e.g. NICER for PSR J0740+6620) rather than a mass-only likelihood — the NICER term is evaluated from the joint M-R posterior, not a truncated Gaussian in mass alone.
- The mass distribution has no EOS-dependent upper bound, so there is no truncation at M_TOV and an ordinary (untruncated) Gaussian likelihood suffices.
- You are sampling GW/kilonova event parameters rather than computing per-EOS prior weights in the NMMA EOS-constraint workflow.

## Inputs

- Candidate EOS sample file (e.g. loaded with numpy/pandas) with per-EOS mass-radius curves and maximum mass M_TOV
- Gaussian mass-posterior parameters (mean, sigma) for PSR J0348+0432 and PSR J1614-2230 radio-timing mass measurements
- NICER joint mass-radius posterior samples/density for PSR J0740+6620 (for the joint M-R likelihood term)

## Outputs

- eos_prior_loglike.csv with one log-likelihood value per candidate EOS (sum of pulsar mass-only and NICER M-R terms)

## How to apply

Load the candidate EOS sample (each EOS with its mass-radius curve and maximum mass M_TOV) using numpy/pandas. For each pulsar with a Gaussian mass posterior (PSR J0348+0432 and PSR J1614-2230), evaluate the probability of observing that mass given the EOS by integrating the Gaussian posterior up to M_TOV via the truncated-Gaussian CDF computed with the error function (erf); EOSs whose M_TOV falls below the low-mass tail of the posterior are thereby heavily suppressed. Implement this through the bilby Likelihood interface used by NMMA, since NMMA uses bilby as its backend. Optionally add the NICER joint mass-radius likelihood for PSR J0740+6620 by evaluating the joint M-R posterior at the radius each candidate EOS predicts at the pulsar's measured mass, and sum the log-likelihood terms per EOS. The physical rationale is that heavy-pulsar masses probe densities beyond those probed by inspiral GWs and supply a lower bound on the maximum neutron-star mass.

## Related tools

- **NMMA** (Framework within which the per-EOS pulsar prior likelihood is implemented and combined with GW/kilonova/EM analyses to constrain the neutron-star EOS) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend whose Likelihood interface is used to implement the mass-only pulsar likelihood)

## Examples

```
from scipy.special import erf; import numpy as np; p_trunc = 0.5*(erf((M_TOV - mu)/(sigma*np.sqrt(2))) - erf((m_min - mu)/(sigma*np.sqrt(2))))  # truncated-Gaussian CDF up to EOS M_TOV
```

## Evaluation signals

- All evaluated per-EOS log-likelihoods are finite (no NaN/inf in eos_prior_loglike.csv).
- Every EOS whose M_TOV falls below the low-mass tail of the pulsar mass posteriors receives a heavily suppressed (large negative) log-likelihood.
- The resulting EOS constraints are consistent with the published behaviour, e.g. R_1.4 = 11.98 (+0.35/-0.40) km.
- Likelihoods are exposed through the bilby Likelihood interface so they can be combined with other NMMA likelihood terms in a single sampler run (dynesty/PyMultiNest/UltraNest).

## Limitations

- The truncation makes the likelihood EOS-dependent through M_TOV, so EOSs with maximum mass below the low-mass tail of the pulsar posteriors are effectively excluded rather than partially down-weighted.
- The mass-only term alone does not constrain radii; it provides a lower bound on the maximum neutron-star mass and must be combined (e.g. with NICER M-R and GW information) for EOS radius constraints.
- No version number, repository URL, or DOI for the publicly available NMMA framework is given in the discussion text, so pinning the exact implementation requires checking the repository.

## Evidence

- [abstract] R_1.4 = 11.98 (+0.35/-0.40) km: [evidence span withheld — anchor status: not_found]
- [discussion] pulsar masses supply a lower bound on maximum NS mass: "This framework allows for the simultaneous analysis of GW and EM signals such as kilonovae and GRB afterglows."
- [readme] bilby is NMMA's backend: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
- [discussion] NMMA incorporates nuclear-physics and radio/X-ray NS measurements: "In addition, our framework allows us to incorporate constraints from nuclear-physics"
