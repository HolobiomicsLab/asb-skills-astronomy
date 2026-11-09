---
name: bayesian-mcmc-posterior-sampling
description: Use when a 1–3 parameter New Physics or cosmology model must be constrained
  by Bayesian posterior sampling against primordial abundances (Y_p, D/H) with Gaussian
  priors, and credible contours or upper bounds are the deliverable.
when_to_use_negative:
- The analysis requires full-chain BSM thermal-history computation beyond few-parameter
  weak-rate modifications — PRyMordial alternatives (PArthENoPE, PRIMAT, AlterBBN,
  NUDEC_BSM) may be more appropriate depending on the model features needed.
- The goal is a single point prediction of BBN abundances under the Standard Model
  rather than a posterior bound (use the PRyMordial SM demo, e.g. PRyMdemoSM, instead
  of the MCMC workflow).
tools:
- name: PRyMordial
  role: 'Fast-precision BBN forward model: predicts Y_p and D/H abundances for given
    weak n↔p rate modifications (NP_delta_nTOp), with small networks and NACRE II
    rates'
  repo: github.com/vallima/PRyMordial
- name: emcee
  role: Affine-invariant ensemble MCMC sampler used to explore the NP_delta_nTOp posterior
    and build 68%/95% credible contours
- name: Python 3
  role: Runtime environment for PRyMordial, the likelihood construction, emcee sampling,
    and matplotlib corner-plot rendering
  repo: github.com/vallima/PRyMordial
- name: Julia
  role: 'Optional backend: SciML/Julia routines can accelerate PRyMordial''s thermal-background
    and rate computations feeding the likelihood'
  repo: github.com/vallima/PRyMordial
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:b4acaf31258f649ee072adbc83fa99cd6df970cf5b24ecad11ba0e24e7b9e6ce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - PRyMordial
  - Python 3
  - emcee
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bayesian-mcmc-posterior-sampling

## Summary

Runs an emcee affine-invariant ensemble MCMC over BBN observables computed with PRyMordial to obtain posterior credible regions on few-parameter New Physics inputs, such as NP_delta_nTOp modifications of the weak n↔p conversion rates, and reports credible bounds against PDG percent-level measurements.

## When to use

Use when a 1–3 parameter New Physics or cosmology model must be constrained by Bayesian posterior sampling against primordial abundances (Y_p, D/H) with Gaussian priors, and credible contours or upper bounds are the deliverable.

## When NOT to use

- The analysis requires full-chain BSM thermal-history computation beyond few-parameter weak-rate modifications — PRyMordial alternatives (PArthENoPE, PRIMAT, AlterBBN, NUDEC_BSM) may be more appropriate depending on the model features needed.
- The goal is a single point prediction of BBN abundances under the Standard Model rather than a posterior bound (use the PRyMordial SM demo, e.g. PRyMdemoSM, instead of the MCMC workflow).

## Inputs

- PRyMordial n↔p conversion-rate prediction pipeline (cloned from github.com/vallima/PRyMordial) with smallnet_flag and nacreii_flags enabled
- PDG measurements of primordial helium-4 (Y_p) and deuterium (D/H) abundances
- Gaussian priors on neutron lifetime tau_n and Omega_B h^2
- Sampling configuration: number of emcee walkers, burn-in steps, and the 2-parameter target NP_delta_nTOp

## Outputs

- Posterior chain (emcee samples for NP_delta_nTOp) saved to disk
- Corner plot with joint 68% and 95% credible contours (matplotlib)
- Upper credible bound on NP_delta_nTOp converted to a few-percent upper limit on the NP modification of the n↔p Born rates

## How to apply

Build a likelihood comparing PRyMordial-predicted light-element abundances (helium-4 mass fraction Y_p and deuterium D/H) against PDG percent-level measurements, with Gaussian priors on tau_n and Omega_B h^2. Enable PRyMordial's smallnet_flag and nacreii_flags so the Born weak rates use the small networks and the NACRE II reaction library. Sample the model parameters (e.g. NP_delta_nTOp) with the emcee affine-invariant ensemble sampler, running enough walkers and burn-in steps to converge the 2-parameter posterior. Compute joint 68% and 95% credible contours from the posterior samples and render them as a matplotlib corner plot; extract the upper credible bound and translate it into the few-percent upper limit on the NP modification of the n↔p Born rates. Persist the posterior chain, corner plot, and upper-bound value to disk.

## Related tools

- **PRyMordial** (Fast-precision BBN forward model: predicts Y_p and D/H abundances for given weak n↔p rate modifications (NP_delta_nTOp), with small networks and NACRE II rates) — github.com/vallima/PRyMordial
- **emcee** (Affine-invariant ensemble MCMC sampler used to explore the NP_delta_nTOp posterior and build 68%/95% credible contours)
- **Python 3** (Runtime environment for PRyMordial, the likelihood construction, emcee sampling, and matplotlib corner-plot rendering) — github.com/vallima/PRyMordial
- **Julia** (Optional backend: SciML/Julia routines can accelerate PRyMordial's thermal-background and rate computations feeding the likelihood) — github.com/vallima/PRyMordial

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The reproduced 68% and 95% joint credible contours for NP_delta_nTOp match the paper's reported contour regions.
- The extracted upper bound on NP_delta_nTOp corresponds to a few-percent (at most) modification of the standard Born n↔p rates.
- The emcee chain shows converged walkers after burn-in (stable 2-parameter posterior).
- The corner plot renders two credible levels (68% and 95%) from the posterior samples with matplotlib.
- Posterior chain, corner plot, and the upper-bound value are all saved to disk as deliverables.

## Limitations

- PRyMordial's precision depends on optional dependencies: Numba, Numdifftools, and Vegas are recommended for best performance, and the code avoids them by default only at the cost of speed.
- The Julia/SciML acceleration path is optional and requires installing Julia and diffeqpy/PyJulia; analyses can run without it but the README positions these as recommended for the best possible usage.
- The bound is limited by the Gaussian priors on tau_n and Omega_B h^2 and by PDG percent-level measurement precision, so conclusions are only as good as those inputs.
- Existing public BBN codes (PArthENoPE, PRIMAT, AlterBBN) suffer from limitations and missing features including lack of a precision BSM-capable treatment — PRyMordial was built to fill these gaps, but users should verify flags (smallnet_flag, nacreii_flags) are set consistently with the paper.

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [abstract] PRyMordial is suitable for state-of-the-art analyses in the Standard Model as well as for general investigations into New Physics active during BBN.: "PRyMordial is suitable for state-of-the-art analyses in the Standard Model as well as for general investigations into New Physics active during BBN."
- [intro] the study of the Early Universe around the BBN epoch offers unique insight on New Physics (NP): "the study of the Early Universe around the BBN epoch offers unique insight on New Physics (NP)"
- [readme] To run a quick example, open the terminal and once in PRyMordial folder just type: python runPRyM_julia.py: "To run a quick example, open the terminal and once in PRyMordial folder just type:"
