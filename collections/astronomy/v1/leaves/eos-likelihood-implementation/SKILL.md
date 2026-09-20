---
name: eos-likelihood-implementation
description: Use when you have a set of candidate EOSs each with a mass-radius curve
  and maximum mass M_TOV, and need to weight them by radio-timing pulsar mass posteriors
  and NICER M-R constraints as part of a Bayesian EOS-inference run.
when_to_use_negative:
- The analysis target is GW/kilonova/GRB afterglow parameter estimation (e.g. GW170817
  light-curve fitting) rather than prior-weighting of a discrete EOS sample.
- You need a continuous EOS-parameter posterior rather than per-candidate-EOS weights;
  the pulsar/NICER terms here are evaluated per candidate EOS, not over EOS microphysical
  parameters.
- No EOS mass-radius curves are available — the likelihoods require R(m) and M_TOV
  for each candidate EOS to evaluate the truncation and the M-R posterior density.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3452
tools:
- name: NMMA
  role: Host framework whose EOS-inference pipeline incorporates pulsar and NICER
    constraints alongside GW and EM data; the likelihood terms are written to plug
    into NMMA's analysis
  repo: github.com/nuclear-multimessenger-astronomy/nmma
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
  source_content_hash: sha256:2d92338875a6a5606ef3e0f29e887e3aab05ee774d4289d06ca1f8c2781df446
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NMMA
  - bilby
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# eos-likelihood-implementation

## Summary

Implements prior-weighting likelihoods for candidate neutron-star equation-of-state (EOS) samples within the NMMA framework: a Gaussian mass-only pulsar likelihood for PSR J0348+0432 and PSR J1614-2230 (truncated at each EOS's maximum mass M_TOV) plus a joint NICER mass-radius likelihood for PSR J0740+6620, so heavy-pulsar masses can constrain the EOS at densities beyond those probed by inspiral gravitational waves.

## When to use

Use when you have a set of candidate EOSs each with a mass-radius curve and maximum mass M_TOV, and need to weight them by radio-timing pulsar mass posteriors and NICER M-R constraints as part of a Bayesian EOS-inference run.

## When NOT to use

- The analysis target is GW/kilonova/GRB afterglow parameter estimation (e.g. GW170817 light-curve fitting) rather than prior-weighting of a discrete EOS sample.
- You need a continuous EOS-parameter posterior rather than per-candidate-EOS weights; the pulsar/NICER terms here are evaluated per candidate EOS, not over EOS microphysical parameters.
- No EOS mass-radius curves are available — the likelihoods require R(m) and M_TOV for each candidate EOS to evaluate the truncation and the M-R posterior density.

## Inputs

- Candidate EOS sample file: per-EOS mass-radius curves and maximum mass M_TOV (loaded via numpy/pandas)
- Gaussian mass posteriors for PSR J0348+0432 and PSR J1614-2230 (radio timing mass measurements)
- Joint M-R posterior for PSR J0740+6620 (NICER mass-radius measurement)

## Outputs

- eos_prior_loglike.csv with per-EOS log-likelihood values combining the pulsar mass and NICER M-R terms

## How to apply

Load the candidate EOS sample (each EOS with its M-R curve and M_TOV) with numpy/pandas. Implement the mass-only pulsar likelihood through the bilby Likelihood interface used by NMMA: for PSR J0348+0432 and PSR J1614-2230, evaluate the Gaussian mass posteriors integrated over neutron-star masses up to the EOS's M_TOV using the error function (truncated-Gaussian CDF). Implement the NICER joint mass-radius likelihood for PSR J0740+6620 by evaluating the joint M-R posterior density at the radius each candidate EOS predicts at the pulsar's measured mass. Sum both log-likelihood terms per EOS and write the per-EOS values to eos_prior_loglike.csv; EOSs whose M_TOV falls below the low-mass tail of the pulsar mass posteriors must receive a heavily suppressed (large negative) log-likelihood, since heavy ~2 solar-mass pulsars also supply a lower bound on the maximum neutron-star mass.

## Related tools

- **NMMA** (Host framework whose EOS-inference pipeline incorporates pulsar and NICER constraints alongside GW and EM data; the likelihood terms are written to plug into NMMA's analysis) — github.com/nuclear-multimessenger-astronomy/nmma
- **bilby** (Backend providing the Likelihood interface through which the mass-only pulsar likelihood is implemented)

## Examples

```
from scipy.special import erf; loglike = 0.5*np.log(2*np.pi*sigma**2) + np.log(0.5*(1+erf((M_TOV-mu)/sigma/np.sqrt(2)))) - np.log(0.5*(1+erf((Mmax-mu)/sigma/np.sqrt(2))))  # truncated-Gaussian pulsar mass likelihood per EOS
```

## Evaluation signals

- All computed per-EOS log-likelihoods written to eos_prior_loglike.csv are finite.
- Every EOS whose M_TOV falls below the low-mass tail of the PSR J0348+0432 / PSR J1614-2230 mass posteriors receives a heavily suppressed (large negative) log-likelihood.
- Combined constraints reproduce the published EOS-constraint behaviour, e.g. R_1.4 = 11.98 (+0.35/-0.40) km.
- The pulsar likelihood is implemented through the bilby Likelihood interface, so it composes with NMMA's standard sampling backends (dynesty/PyMultiNest/UltraNest).

## Limitations

- Radio pulsar masses probe EOS information at densities beyond inspiral gravitational waves but act mainly through the prior likelihood (including a lower bound on maximum mass); they do not by themselves measure radii.
- The truncation at M_TOV means an EOS sample that is too soft to support ~2 solar-mass pulsars is effectively ruled out rather than finely ranked.

## Evidence

- [other] Radio mass measurements of ~2 solar-mass pulsars provide EOS information at densities beyond those probed by inspiral gravitational waves, and additionally supply a lower bound on the maximum neutron-star mass: [evidence span withheld — anchor status: not_found]
- [abstract] Incorporating all available information, we estimate the radius of a 1.4M⊙ neutron star to be R=11.98−0.40+0.35 km.: [evidence span withheld — anchor status: not_found]
- [readme] Since NMMA uses bilby as its backend, please also cite the bilby companion paper: "Since NMMA uses bilby as its backend, please also cite the bilby companion paper"
- [discussion] our framework allows us to incorporate constraints from nuclear-physics: "This framework allows for the simultaneous analysis of GW and EM signals such as kilonovae and GRB afterglows. In addition, our framework allows us to incorporate constraints from nuclear-physics"
