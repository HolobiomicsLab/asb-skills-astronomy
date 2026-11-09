---
name: nuclear-reaction-network-configuration
description: Use when a task needs PRyMordial's BBN abundance predictions with configurable
  weak n↔p Born rates (e.g. enabling smallnet_flag and nacreii_flags before a Bayesian
  NP_delta_nTOp constraint with emcee).
when_to_use_negative:
- The analysis requires a full BSM thermal-bath/decoupling treatment of the kind handled
  by standalone codes like NUDEC_BSM or PRIMAT rather than configuring PRyMordial's
  n↔p network.
- The task is pure Monte Carlo posterior sampling (e.g. emcee walker/burn-in tuning)
  with no need to change the underlying nuclear network or reaction-rate configuration.
- Only pre-computed BBN abundances are needed for a standard cosmological likelihood
  — in that case use the provided PRyM_Yp_DH_cosmoMC_2023.dat table directly instead
  of re-configuring the network.
edam_operation: http://edamontology.org/operation_3799
edam_topics: []
tools:
- name: PRyMordial
  role: Core BBN computation package whose configuration flags (smallnet_flag, nacreii_flags)
    select the small networks and NACRE II library for the Born n↔p rates
  repo: https://github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which the package is written and runs entirely; used to execute
    runPRyM_julia.py and the demo notebooks
- name: Julia
  role: Optional advanced-user backend via the SciML kit and diffeqpy for faster thermal-background
    routines
- name: NumPy/SciPy/Numba
  role: Mandatory (NumPy, SciPy) and recommended (Numba) Python dependencies for running
    the reaction network efficiently
- name: emcee
  role: Downstream affine-invariant ensemble sampler that consumes the configured
    pipeline when constraining NP_delta_nTOp against Y_p and D/H
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:7b74ce88c385acc5fce00c5c100026d3bd96be6b1ef26198dd7433e30a2a6a1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - Julia
  - NumPy/SciPy/Numba
  - PRyMordial
  - Python 3
  - emcee
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# nuclear-reaction-network-configuration

## Summary

Configure PRyMordial's Big Bang Nucleosynthesis pipeline — in particular its neutron-to-proton (n↔p) weak conversion rates — by enabling the smallnet_flag and nacreii_flags so the Born rates run over the small nuclear networks with the NACRE II reaction library, yielding fast, precise Y_p and D/H predictions for Standard Model or New Physics (NP) studies.

## When to use

Use when a task needs PRyMordial's BBN abundance predictions with configurable weak n↔p Born rates (e.g. enabling smallnet_flag and nacreii_flags before a Bayesian NP_delta_nTOp constraint with emcee).

## When NOT to use

- The analysis requires a full BSM thermal-bath/decoupling treatment of the kind handled by standalone codes like NUDEC_BSM or PRIMAT rather than configuring PRyMordial's n↔p network.
- The task is pure Monte Carlo posterior sampling (e.g. emcee walker/burn-in tuning) with no need to change the underlying nuclear network or reaction-rate configuration.
- Only pre-computed BBN abundances are needed for a standard cosmological likelihood — in that case use the provided PRyM_Yp_DH_cosmoMC_2023.dat table directly instead of re-configuring the network.

## Inputs

- PRyMordial repository clone (github.com/vallima/PRyMordial)
- Pipeline configuration flags (smallnet_flag, nacreii_flags) controlling Born-rate network and reaction-library selection
- Physical/prior parameters such as tau_n and Omega_B h^2 for downstream likelihood use
- Optional: PRyM_Yp_DH_cosmoMC_2023.dat numerical table for standard cosmological BBN likelihoods

## Outputs

- PRyMordial-predicted BBN light-element abundances (helium-4 Y_p, deuterium D/H)
- Effective number of relativistic degrees of freedom (N_eff) including non-instantaneous decoupling effects
- Configured Born n↔p conversion rates feeding NP_delta_nTOp-style parameter studies

## How to apply

Clone github.com/vallima/PRyMordial and run the pipeline with `python runPRyM_julia.py`, or follow the PRyMdemoSM and PRyMdemoNP Jupyter notebooks for guided usage. Enable smallnet_flag and nacreii_flags so the Born n↔p conversion rates are computed over the small nuclear networks using the NACRE II thermonuclear reaction library, which is the configuration used when constraining NP modifications (NP_delta_nTOp) of the weak rates against PDG helium-4 (Y_p) and deuterium (D/H) measurements. Set Gaussian priors on tau_n and Omega_B h^2 if the configured network feeds a likelihood, and keep the code's optional Numba/Numdifftools dependencies installed for the best numerical performance, with the PyJulia/diffeqpy route as an optional switch to the Julia SciML kit. The code outputs light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects; a dedicated numerical table for standard cosmological analyses (PRyM_Yp_DH_cosmoMC_2023.dat) is provided for BBN likelihood interconnection.

## Related tools

- **PRyMordial** (Core BBN computation package whose configuration flags (smallnet_flag, nacreii_flags) select the small networks and NACRE II library for the Born n↔p rates) — https://github.com/vallima/PRyMordial
- **Python 3** (Language in which the package is written and runs entirely; used to execute runPRyM_julia.py and the demo notebooks)
- **Julia** (Optional advanced-user backend via the SciML kit and diffeqpy for faster thermal-background routines)
- **NumPy/SciPy/Numba** (Mandatory (NumPy, SciPy) and recommended (Numba) Python dependencies for running the reaction network efficiently)
- **emcee** (Downstream affine-invariant ensemble sampler that consumes the configured pipeline when constraining NP_delta_nTOp against Y_p and D/H)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Running `python runPRyM_julia.py` from the PRyMordial folder completes the quick example without missing-dependency errors (NumPy and SciPy mandatory; Numba, Numdifftools, Vegas recommended).
- With nacreii_flags enabled, Born rates reflect the NACRE II reaction library; the resulting Y_p and D/H should be consistent with the PDG percent-level measurements used in the paper's Bayesian analysis.
- Predicted abundances and N_eff include non-instantaneous decoupling effects and reproduce the values tabulated in PRyM_Yp_DH_cosmoMC_2023.dat for the Standard Model configuration.
- When configured for the NP study, the few-percent bound on NP modifications of the n↔p Born rates is recoverable, i.e. the 68%/95% contours on NP_delta_nTOp match the paper's reported joint credible regions.
- PRyMdemoSM and PRyMdemoNP notebooks execute end-to-end, confirming the Standard Model and New Physics demonstration paths both run under the chosen configuration.

## Limitations

- NumPy and SciPy are mandatory dependencies; Numba, Numdifftools, and Vegas are recommended but the code can avoid Numba and Numdifftools and does not use Vegas by default — performance may degrade without them.
- The Julia/SciML route is optional and requires installing PyJulia and diffeqpy plus the Julia language itself; users who skip it lose the optional switch to SciML routines.
- Repository documentation provides no changelog/discussion content; authors and licensing details live in doc/CREDITS, doc/COPYING, and doc/LICENSE rather than the README.

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [abstract] The package is written in Python, but more advanced users can optionally take advantage of the open-source community for Julia.: "The package is written in Python, but more advanced users can optionally take advantage of the open-source community for Julia."
- [readme] README quick-start invocation: "To run a quick example, open the terminal and once in PRyMordial folder just type: 

python runPRyM_julia.py"
- [readme] README demo notebooks: "Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP."
- [readme] README dependency list: "The code can easily avoid dependencies on Numba and Numdifftools and it does not make use of Vegas by default."
