---
name: power-spectrum-suppression-modelling
description: Use when a CDM linear power spectrum/transfer function must be filtered
  to mimic warm dark matter (parameterised by m_X, g_X, Ω_X, h), e.g. to predict fewer
  low-mass haloes than CDM in an HMF calculation.
when_to_use_negative:
- Input model is pure CDM or another cold model — no small-scale suppression filter
  should be applied
- A first-principles WDM transfer function from a Boltzmann solver is required rather
  than the analytic Bode et al. (2001) fitting-formula filter
- The supplied transfer-function file is not produced by CAMB or in CAMB format
tools:
- name: hmf
  role: Python engine whose alternatives.wdm module style hosts the WDM filter and
    whose k-grid, spline interpolation and P(k) ← P·T_WDM^2 workflow the filter plugs
    into
  repo: https://github.com/halomod/hmf
- name: CAMB (pycamb)
  role: Computes the underlying CDM transfer functions T(k) that the WDM filter is
    applied to
  repo: https://github.com/joezuntz/pycamb
- name: HMFcalc
  role: Reference web frontend exposing the WDM model selection and parameter combinations
    (legends labelled by parameter combination); used to cross-check the filtered
    HMF
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:c97a29b906c2d9d94a6a01395cefdb9be48d0a0b378295bcfa37201024b6702e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB (pycamb)
  - HMFcalc
  - Python
  - hmf
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# power-spectrum-suppression-modelling

## Summary

Implements the Bode et al. (2001) warm-dark-matter transfer function filter T_X(k) = (1 + (αk)^{2ν})^{-5/ν}, with ν = 1.2 and α from Eq 15, to suppress small-scale power in a CDM power spectrum before computing the halo mass function as HMFcalc does.

## When to use

Use when a CDM linear power spectrum/transfer function must be filtered to mimic warm dark matter (parameterised by m_X, g_X, Ω_X, h), e.g. to predict fewer low-mass haloes than CDM in an HMF calculation.

## When NOT to use

- Input model is pure CDM or another cold model — no small-scale suppression filter should be applied
- A first-principles WDM transfer function from a Boltzmann solver is required rather than the analytic Bode et al. (2001) fitting-formula filter
- The supplied transfer-function file is not produced by CAMB or in CAMB format

## Inputs

- CDM linear transfer function T(k) from CAMB (pycamb.transfers output) or a CAMB-format file
- Cosmological parameters: Ω_X (WDM density), h, m_X (keV), g_X (degrees of freedom, fiducial 1.5)
- Shared wavenumber grid k (h/Mpc), default hmf range ~1e-8 to 2000 h/Mpc

## Outputs

- WDM transfer function T_WDM(k) = T_CDM(k) · [1 + (αk)^{2ν}]^{-5/ν} evaluated on the k-grid
- Filtered power spectrum P(k) ← P(k) · T_WDM(k)^2 (pre-σ_8 normalisation)
- Table of k and T_X(k) per m_X with labelled columns (e.g. wdm_transfer_functions.csv)

## How to apply

Obtain the CDM linear transfer function T(k) from CAMB (via pycamb, or import a CAMB-format file), interpolated with linear splines in log-space over the hmf k-range (kmin ~1e-8 to kmax ~2000 h/Mpc). For each WDM mass m_X compute α from Ω_X, h, m_X and g_X via Eqs 14-15, holding ν = 1.2 and g_X = 1.5 fixed at their fiducial single-species values so m_X is the only free parameter. Apply the filter as T_WDM(k) = [1 + (αk)^{2ν}]^{-5/ν} and modify the power spectrum as P(k) ← P(k) · T_WDM(k)^2 before normalising to σ_8. Evaluate T_X(k) on a shared k-grid (e.g. m_X = 0.5, 1.0, 2.0 keV), tabulate with labelled columns per parameter combination, and validate ordering and limits as in evaluation signals.

## Related tools

- **hmf** (Python engine whose alternatives.wdm module style hosts the WDM filter and whose k-grid, spline interpolation and P(k) ← P·T_WDM^2 workflow the filter plugs into) — https://github.com/halomod/hmf
- **CAMB (pycamb)** (Computes the underlying CDM transfer functions T(k) that the WDM filter is applied to) — https://github.com/joezuntz/pycamb
- **HMFcalc** (Reference web frontend exposing the WDM model selection and parameter combinations (legends labelled by parameter combination); used to cross-check the filtered HMF)

## Examples

```
T_wdm = T_cdm * (1 + (alpha*k)**(2*1.2))**(-5/1.2)  # Bode+2001 Eq 14, alpha from Eq 15 with g_X=1.5
```

## Evaluation signals

- T_WDM(k) → 1 as k → 0 (no suppression of large-scale power)
- Suppression grows monotonically at high k for every m_X
- Suppression is ordered by mass: smaller m_X gives stronger suppression, consistent with viable WDM models predicting fewer low-mass haloes than CDM
- α reproduces Eq 15 given Ω_X, h, m_X and fixed g_X = 1.5, ν = 1.2 (m_X is the only free parameter)
- Power spectrum is modified as P(k) ← P(k) · T_WDM(k)^2 (square, not first power) before σ_8 normalisation

## Limitations

- ν and g_X are fixed at fiducial values (g_X = 1.5, single species); only m_X varies, so non-fiducial species/DOF combinations need code changes
- Transfer-function input must be CAMB-produced or in CAMB format; high-k coverage may require extrapolate_with_eh=True (Eisenstein & Hu fitting formula)
- pycamb is deprecated: 'Since camb itself now ships with python bindings you should use those instead' — the joezuntz/pycamb README warns the code no longer works with recent CAMB versions
- Interpolation uses linear splines on the transfer function in log-space, so accuracy degrades outside the original k-bounds

## Evidence

- [intro] WDM vs CDM halo abundance motivation: [evidence span withheld — anchor status: not_found]
