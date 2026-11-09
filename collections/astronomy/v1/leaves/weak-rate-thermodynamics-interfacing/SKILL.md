---
name: weak-rate-thermodynamics-interfacing
description: Use when computing BBN observables that require the thermal history (e.g.
  neutron-to-proton conversion around MeV-scale decoupling) to be coupled to a nuclear
  reaction network rather than assumed instantaneous.
when_to_use_negative:
- The analysis targets nuclear reaction network details (e.g. NACRE II rate sensitivities)
  rather than the thermal/weak-rate coupling stage
- A sudden-decoupling approximation is acceptable and non-instantaneous weak-rate
  treatment is unnecessary
- The task is a likelihood fit using the precomputed PRyM_Yp_DH_cosmoMC_2023.dat table
  instead of a fresh thermodynamics run
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: PRyMordial
  role: Package providing the thermal-background and weak-rates computation with non-instantaneous
    decoupling, to be interfaced with the nuclear network
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Mandatory runtime for PRyMordial's thermodynamics and weak-rate computation
- name: SciML kit
  role: Optional Julia routines used to speed up the differential-equation integration
    behind the thermal/weak-rate stage
- name: Julia
  role: Optional language backend (via PyJulia and diffeqpy) for accelerated BBN-era
    differential equations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:307b739b0287e733f43a1fb04b7a3ee5cf9a23a43b935ba306f0f07d0603af06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - PRyMordial
  - Python 3
  - SciML kit
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# weak-rate-thermodynamics-interfacing

## Summary

Interconnect PRyMordial's first-principle thermal-bath computation with ab-initio neutron-to-proton weak-rate conversion, using the stored thermal-background and weak-rates arrays that include non-instantaneous decoupling effects, as the upstream stage for BBN nuclear-network abundance runs.

## When to use

Use when computing BBN observables that require the thermal history (e.g. neutron-to-proton conversion around MeV-scale decoupling) to be coupled to a nuclear reaction network rather than assumed instantaneous.

## When NOT to use

- The analysis targets nuclear reaction network details (e.g. NACRE II rate sensitivities) rather than the thermal/weak-rate coupling stage
- A sudden-decoupling approximation is acceptable and non-instantaneous weak-rate treatment is unnecessary
- The task is a likelihood fit using the precomputed PRyM_Yp_DH_cosmoMC_2023.dat table instead of a fresh thermodynamics run

## Inputs

- PRyMordial thermal-background arrays from the upstream thermodynamics run
- Weak-rates arrays for neutron-to-proton conversion (non-instantaneous decoupling treatment)
- Python 3 environment with NumPy and SciPy installed (mandatory)

## Outputs

- Interfaced thermal/weak-rate arrays ready to feed the nuclear-abundance ODE integration
- Effective number of relativistic degrees of freedom (N_eff) consistent with the non-instantaneous decoupling treatment
- Inputs for downstream BBN abundance extraction (e.g. the res array with Y_P, D/H, 3He/H, 7Li/H)

## How to apply

Load the stored thermal-background and weak-rates arrays produced by the upstream PRyMordial thermodynamics run, which treats neutron-proton decoupling non-instantaneously rather than with a sudden-decoupling approximation. Ensure NumPy and SciPy (mandatory dependencies) are installed, and optionally enable Numba or the Julia SciML kit routines (via PyJulia and diffeqpy) for faster integration. Interface the weak rates with the nuclear network over the BBN temperature range from O(10) MeV down to O(1) keV, so the neutron-to-proton conversion feeds the computed abundances. Decide whether the quick Julia path (runPRyM_julia.py) or the pure Python 3 path is needed depending on whether Monte Carlo-scale exploration is planned.

## Related tools

- **PRyMordial** (Package providing the thermal-background and weak-rates computation with non-instantaneous decoupling, to be interfaced with the nuclear network) — github.com/vallima/PRyMordial
- **Python 3** (Mandatory runtime for PRyMordial's thermodynamics and weak-rate computation)
- **SciML kit** (Optional Julia routines used to speed up the differential-equation integration behind the thermal/weak-rate stage)
- **Julia** (Optional language backend (via PyJulia and diffeqpy) for accelerated BBN-era differential equations)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The stored thermal-background and weak-rates arrays are successfully loaded and available before the nuclear network is initialized
- The computed N_eff reflects non-instantaneous decoupling effects, as the code promises 'precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom'
- The temperature integration spans from O(10) MeV down to O(1) keV as in the article's workflow
- Running with the optional Julia/SciML path (runPRyM_julia.py) yields results consistent with the pure Python 3 path
- Downstream abundance output (bbn_abundances.csv / res array) is only produced if the weak-rate interfacing stage completed without error

## Limitations

- Numba, Numdifftools, Vegas, PyJulia, and diffeqpy are optional; the code 'can easily avoid dependencies on Numba and Numdifftools and it does not make use of Vegas by default', but installing them is 'recommended for the best possible usage of the package'
- The Julia path requires a Julia installation and the SciML scientific machine learning software
- No discussion-section limitations or reproducibility gaps were extractable from the article context ('No changelog found.')

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [intro] interconnect first-principle thermal background with ab-initio neutron-to-proton conversion calculations: [evidence span withheld — anchor status: not_found]
- [intro] optional switch to some routines of the SciML kit: "the optional switch to some routines of the SciML kit [58], the open-source software for scientific machine learning in Julia"
- [other] Load the stored thermal-background and weak-rates arrays produced by the upstream PRyMordial thermodynamics run: [evidence span withheld — anchor status: not_found]
- [readme] To run a quick example, open the terminal and once in PRyMordial folder just type: python runPRyM_julia.py: "To run a quick example, open the terminal and once in PRyMordial folder just type: python runPRyM_julia.py"
- [readme] NumPy (mandatory) – pip install numpy: "- NumPy (mandatory) – pip install numpy"
