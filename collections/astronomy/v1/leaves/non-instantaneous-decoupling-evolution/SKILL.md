---
name: non-instantaneous-decoupling-evolution
description: Use when a precision BBN analysis requires N_eff and neutron-to-proton
  conversion rates computed from a coupled, non-instantaneous neutrino-decoupling
  thermal history, optionally with new-physics energy exchange (NP_e_flag/NP_nu_flag)
  with the plasma or neutrino bath.
when_to_use_negative:
- The analysis only needs an instantaneous-decoupling or fixed-N_eff approximation
  (e.g. quick parameter scans where percent-level decoupling corrections are irrelevant).
- The analysis concerns BBN abundance network details or thermonuclear rate uncertainties
  rather than the thermal history — PRyMordial handles those as separate stages.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3450
- http://edamontology.org/topic_0221
tools:
- name: PRyMordial
  role: 'Main package: solves the coupled thermal-history ODEs with non-instantaneous
    neutrino decoupling, with NP_e_flag/NP_nu_flag switches for new-physics energy
    exchange, and passes N_eff and the n-to-p history to the BBN abundance computation.'
  repo: https://github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which the thermal-history evolution runs entirely; NumPy and SciPy
    are mandatory dependencies, Numba and Numdifftools recommended for performance.
- name: Julia / SciML kit
  role: 'Optional backend: PyJulia and diffeqpy allow switching the ODE integration
    routines to the SciML scientific-machine-learning kit for faster evolution.'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:2f857e1e9801c4d5353872e10eabdbe55494ea9291cda02cb20b47462db83a08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - Julia / SciML kit
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3450
  - http://edamontology.org/topic_0221
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# non-instantaneous-decoupling-evolution

## Summary

Evolve the coupled electron-photon plasma and neutrino thermal history with PRyMordial's ODE solver while retaining non-instantaneous neutrino decoupling effects, yielding N_eff and the neutron-to-proton conversion history feeding BBN abundance computations.

## When to use

Use when a precision BBN analysis requires N_eff and neutron-to-proton conversion rates computed from a coupled, non-instantaneous neutrino-decoupling thermal history, optionally with new-physics energy exchange (NP_e_flag/NP_nu_flag) with the plasma or neutrino bath.

## When NOT to use

- The analysis only needs an instantaneous-decoupling or fixed-N_eff approximation (e.g. quick parameter scans where percent-level decoupling corrections are irrelevant).
- The analysis concerns BBN abundance network details or thermonuclear rate uncertainties rather than the thermal history — PRyMordial handles those as separate stages.

## Inputs

- PRyMordial run configuration with NP_e_flag and NP_nu_flag settings (on/off and new-species spin/mass parameters)
- Choice of integrator backend: pure Python (NumPy/SciPy, Numba-recommended) or optional Julia SciML/diffeqpy routines
- Thermal-history initial conditions (plasma and neutrino bath temperatures, SM particle content)

## Outputs

- N_eff (effective number of relativistic degrees of freedom) as a function of temperature/time
- Neutron-to-proton conversion history feeding the BBN solver
- Evolved neutrino-to-photon temperature ratio and time-temperature relation
- Input abundances (e.g. Yp, D/H) for BBN likelihood tables such as PRyM_Yp_DH_cosmoMC_2023.dat

## How to apply

Configure PRyMordial's run (Python 3 entry point, e.g. runPRyM_julia.py or the PRyMdemoSM/PRyMdemoNP notebooks) and enable the NP_nu_flag and NP_e_flag switches to model new-physics energy exchange with the neutrino bath and electromagnetic plasma respectively; if no new species is studied, keep these off for the SM history. Evolve the coupled thermodynamic ODE system with the code's integrator (optionally the Julia SciML/diffeqpy routines for speed), deliberately retaining non-instantaneous decoupling effects rather than assuming instantaneous neutrino freeze-out, so that the time-temperature relation and the neutrino-to-photon temperature ratio are solved consistently. From the solution, extract N_eff and the neutron-to-proton conversion history, then feed these into PRyMordial's BBN abundance computation (light-element yields, e.g. Yp and D/H as tabulated in PRyM_Yp_DH_cosmoMC_2023.dat for standard analyses). Validate by comparing N_eff against the expected SM value including non-instantaneous decoupling corrections and by checking that predicted abundances match the paper's SM benchmarks.

## Related tools

- **PRyMordial** (Main package: solves the coupled thermal-history ODEs with non-instantaneous neutrino decoupling, with NP_e_flag/NP_nu_flag switches for new-physics energy exchange, and passes N_eff and the n-to-p history to the BBN abundance computation.) — https://github.com/vallima/PRyMordial
- **Python 3** (Language in which the thermal-history evolution runs entirely; NumPy and SciPy are mandatory dependencies, Numba and Numdifftools recommended for performance.)
- **Julia / SciML kit** (Optional backend: PyJulia and diffeqpy allow switching the ODE integration routines to the SciML scientific-machine-learning kit for faster evolution.)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- N_eff from the SM run matches the published PRyMordial benchmark including non-instantaneous decoupling corrections (not the instantaneous-decoupling value).
- The evolved neutrino-to-photon temperature ratio is consistent with the expected post-decoupling value, and with the paper's Fig. 4 scan behaviour when NP_nu_flag/NP_e_flag are enabled.
- Predicted Yp and D/H from the downstream abundance computation agree with the values in the PRyM_Yp_DH_cosmoMC_2023.dat cosmoMC table for standard settings.
- Pure-Python and optional Julia/SciML integrator runs give consistent thermal histories within numerical tolerance.
- The reproduced abundance bands and overlaid observational exclusion region match the corresponding published figure panel (e.g. Fig. 4 for the new-species scan).

## Limitations

- Full performance benefits require optional dependencies: Numba and Numdifftools are recommended, and the Julia SciML/diffeqpy switch requires installing the Julia language and sciml.ai software.
- The code can avoid Numba and Numdifftools dependencies, but the README states their installation 'remains recommended for the best possible usage of the package'.
- The discussion-section content of the article is absent in the provided context, so no extractable reproducibility gaps or changelog information beyond the paper and README is available.

## Evidence

- [abstract] Fast precise evaluation of BBN light-element abundances together with N_eff, retaining non-instantaneous decoupling effects: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [abstract] PRyMordial suitable for NP active during BBN: "PRyMordial is suitable for state-of-the-art analyses in the Standard Model as well as for general investigations into New Physics active during BBN."
- [intro] Optional switch to Julia SciML routines for the evolution: "the optional switch to some routines of the SciML kit [58], the open-source software for scientific machine learning in Julia"
- [intro] PRyMordial written and runs entirely with Python 3: "The package is written and runs entirely with Python 3"
- [readme] Quick example invocation and example notebooks from the README: "To run a quick example, open the terminal and once in PRyMordial folder just type: python runPRyM_julia.py"
- [readme] Dedicated numerical table for BBN likelihoods: "A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo."
