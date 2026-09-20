---
name: pulsar-mass-posterior-integration
description: Use when a set of candidate equations of state (each with an M-R curve
  and maximum mass M_TOV) must be weighted by pulsar mass posteriors and NICER mass-radius
  data to constrain the neutron-star EOS.
when_to_use_negative:
- The analysis targets GW/kilonova/GRB-afterglow lightcurve inference (e.g. GW170817,
  AT2017gfo) rather than weighting an EOS sample by pulsar mass and NICER posteriors.
- No candidate EOS sample with mass-radius curves and M_TOV values is available —
  the truncated-Gaussian CDF and NICER posterior evaluation both require per-EOS radii
  and M_TOV.
- Pulsar data are full timing residuals rather than published Gaussian mass posteriors;
  the erf-truncated likelihood assumes Gaussian mass posteriors.
tools:
- name: NMMA
  role: Framework in which the EOS-constraint analysis is run; provides the EOS prior-likelihood
    structure and combine-EOS tooling
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
  source_content_hash: sha256:54d815ce1787d7514932fb48ec01b9752a399b37cbd8dde0d01a1f1d8365c6d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - bilby
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# pulsar-mass-posterior-integration

## Summary

Computes per-EOS likelihood contributions from radio timing mass measurements of heavy pulsars (PSR J0348+0432, PSR J1614-2230) and the NICER joint mass-radius posterior for PSR J0740+6620, within the NMMA framework. Heavy pulsar masses probe densities beyond inspiral gravitational waves and impose a lower bound on the maximum neutron-star mass M_TOV.

## When to use

Use when a set of candidate equations of state (each with an M-R curve and maximum mass M_TOV) must be weighted by pulsar mass posteriors and NICER mass-radius data to constrain the neutron-star EOS.

## When NOT to use

- The analysis targets GW/kilonova/GRB-afterglow lightcurve inference (e.g. GW170817, AT2017gfo) rather than weighting an EOS sample by pulsar mass and NICER posteriors.
- No candidate EOS sample with mass-radius curves and M_TOV values is available — the truncated-Gaussian CDF and NICER posterior evaluation both require per-EOS radii and M_TOV.
- Pulsar data are full timing residuals rather than published Gaussian mass posteriors; the erf-truncated likelihood assumes Gaussian mass posteriors.

## Inputs

- Candidate EOS sample table (per-EOS mass-radius curve and maximum mass M_TOV), loadable via numpy/pandas
- Gaussian mass posteriors for PSR J0348+0432 and PSR J1614-2230 (radio timing mass measurements, ~2 solar masses)
- NICER joint mass-radius posterior for PSR J0740+6620

## Outputs

- eos_prior_loglike.csv with per-EOS pulsar-mass and NICER log-likelihood values

## How to apply

Load the candidate EOS sample (each EOS with its mass-radius curve and M_TOV) with numpy/pandas. Implement the mass-only pulsar likelihood through the bilby Likelihood interface used by NMMA: for PSR J0348+0432 and PSR J1614-2230, evaluate the Gaussian mass posteriors integrated over neutron-star masses up to the EOS's M_TOV, i.e. a truncated-Gaussian CDF computed via the error function. For PSR J0740+6620, evaluate the NICER joint M-R posterior density at the radius each EOS predicts at the pulsar's measured mass. Sum both log-likelihood terms for every EOS and write per-EOS values to eos_prior_loglike.csv; EOSs whose M_TOV falls below the low-mass tail of the pulsar mass posteriors must receive a heavily suppressed (large negative) log-likelihood.

## Related tools

- **NMMA** (Framework in which the EOS-constraint analysis is run; provides the EOS prior-likelihood structure and combine-EOS tooling) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend whose Likelihood interface is used to implement the mass-only pulsar likelihood)

## Examples

```
from bilby.core.likelihood import Likelihood; from scipy.special import erf; logL = -0.5*((m-mu)/sig)**2 - np.log(0.5*(1+erf((M_TOV-mu)/(np.sqrt(2)*sig))))  # truncated-Gaussian pulsar mass likelihood per EOS
```

## Evaluation signals

- All evaluated log-likelihoods are finite (no NaN/inf in eos_prior_loglike.csv).
- Every EOS whose M_TOV falls below the low-mass tail of the pulsar mass posteriors receives a heavily suppressed (large negative) log-likelihood.
- Per-EOS log-likelihoods equal the sum of the two terms: the erf-truncated Gaussian pulsar mass likelihood and the NICER joint M-R posterior density evaluated at each EOS's predicted radius at PSR J0740+6620's measured mass.
- When combined with the full NMMA multi-messenger analysis, the joint constraint reproduces the published R_1.4 = 11.98 (+0.35/-0.40) km behaviour.

## Limitations

- The pulsar mass likelihood truncates at each EOS's M_TOV; EOSs with low maximum mass are effectively rejected, so numerical care is needed for the truncated-Gaussian CDF near the truncation boundary.
- The NICER term evaluates a joint M-R posterior only at each EOS's predicted radius at the measured mass — it cannot constrain EOSs that fail to reach the pulsar's mass.
- Radio pulsar masses supply EOS information only at densities beyond those probed by inspiral gravitational waves; they act as a lower bound on M_TOV rather than a full EOS constraint.

## Evidence

- [abstract] R_1.4 result: [evidence span withheld — anchor status: not_found]
- [discussion] framework scope: "This framework allows for the simultaneous analysis of GW and EM signals such as kilonovae and GRB afterglows. In addition, our framework allows us to incorporate constraints from nuclear-physics"
- [readme] bilby backend: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
