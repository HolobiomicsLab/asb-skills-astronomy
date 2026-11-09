---
name: camb-transfer-function-generation
description: Use when a halo-mass-function or power-spectrum calculation needs a linear
  CDM transfer function T(k) from CAMB, or when a warm-DM transfer function must be
  derived by filtering T_CDM(k) with the Bode et al. (2001) form.
when_to_use_negative:
- Only the final halo mass function is needed and no transfer-function file or cosmology
  variation is required — use hmf's built-in defaults rather than re-running CAMB
- pycamb must be used with a recent CAMB release — the joezuntz/pycamb README states
  it no longer works with recent versions of camb and recommends CAMB's own Python
  bindings
- Input transfer function is neither CAMB-produced nor in CAMB format — hmf will reject
  the file
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CAMB
  role: Computes the linear matter transfer functions T(k) used by hmf/HMFcalc
  repo: http://camb.info
- name: pycamb
  role: Python bindings exposing CAMB's transfers routine, which returns k, T_i(k,z)
    and sigma8(z)
  repo: https://github.com/joezuntz/pycamb
- name: hmf
  role: Consumes the CAMB transfer function, interpolates it in log-space, and applies
    the WDM Bode et al. (2001) filter (hmf.alternatives.wdm)
  repo: https://github.com/halomod/hmf
- name: HMFcalc
  role: Web frontend that surfaces the transfer-function and WDM-filter workflow to
    users
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:7d3e57f03708c68f1331dee996dd17199c58cbbe3d11564509444cce35c3202c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CAMB
  - HMFcalc
  - Python
  - hmf
  - pycamb
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# camb-transfer-function-generation

## Summary

Generates linear matter transfer functions T(k) using CAMB (via pycamb or CAMB-format files) as the first stage of the hmf/HMFcalc halo-mass-function pipeline, and supports WDM filtering via the Bode et al. (2001) T_X(k) = (1 + (αk)^{2ν})^{-5/ν} modification with ν = 1.2.

## When to use

Use when a halo-mass-function or power-spectrum calculation needs a linear CDM transfer function T(k) from CAMB, or when a warm-DM transfer function must be derived by filtering T_CDM(k) with the Bode et al. (2001) form.

## When NOT to use

- Only the final halo mass function is needed and no transfer-function file or cosmology variation is required — use hmf's built-in defaults rather than re-running CAMB
- pycamb must be used with a recent CAMB release — the joezuntz/pycamb README states it no longer works with recent versions of camb and recommends CAMB's own Python bindings
- Input transfer function is neither CAMB-produced nor in CAMB format — hmf will reject the file

## Inputs

- Cosmological parameters (H0, omegab, omegac, scalar_index ns, redshift z) for CAMB
- Existing CAMB-format transfer-function file (optional alternative to running CAMB)
- WDM parameters: m_X, g_X (default 1.5), Ω_X, h, ν (default 1.2)
- k-grid bounds of the hmf package (~1e-8 to 2000 h/Mpc)

## Outputs

- Linear matter transfer function T(k) interpolated on the hmf k-grid
- Unnormalised power spectrum P0(k) = k^n T^2 and sigma_8-normalised P(k)
- WDM transfer functions T_X(k) for each m_X parameter combination (e.g. a labelled wdm_transfer_functions.csv)

## How to apply

Obtain T(k) either by calling CAMB (the paper uses the public CAMB code of Lewis et al. 2000, accessed through pycamb, whose transfers routine returns k, T_i(k,z) and sigma8) or by importing a file that was produced by CAMB or is in CAMB format. Interpolate the transfer function with linear splines on the function in log-space and re-grid it between the hmf k-bounds (kmin ~ 1e-8 to kmax ~ 2000 h/Mpc); optionally set extrapolate_with_eh=True to extend to higher k with the Eisenstein & Hu fitting formula. Compute the unnormalised spectrum P0(k) = k^n T^2 and normalise to sigma_8 before applying the growth factor d(z). For WDM, compute α from Ω_X, h, m_X and g_X per Eqs 14-15 (default g_X = 1.5, ν = 1.2 fixed) and multiply the CDM transfer function by [1 + (αk)^{2ν}]^{-1}, then re-normalise the power spectrum. Evaluate T_X(k) for each mass parameter combination (e.g. m_X = 0.5, 1.0, 2.0 keV) on the shared k-grid and label outputs by that combination, as HMFcalc labels its legends.

## Related tools

- **CAMB** (Computes the linear matter transfer functions T(k) used by hmf/HMFcalc) — http://camb.info
- **pycamb** (Python bindings exposing CAMB's transfers routine, which returns k, T_i(k,z) and sigma8(z)) — https://github.com/joezuntz/pycamb
- **hmf** (Consumes the CAMB transfer function, interpolates it in log-space, and applies the WDM Bode et al. (2001) filter (hmf.alternatives.wdm)) — https://github.com/halomod/hmf
- **HMFcalc** (Web frontend that surfaces the transfer-function and WDM-filter workflow to users)

## Examples

```
python -c "import pycamb; T,E,B,X = pycamb.camb(2000, H0=72, omegab=0.04, omegav=0.6, omegac=0.36); k,Ti,s8 = pycamb.transfers()"
```

## Evaluation signals

- T_X(k) -> 1 as k -> 0 for every WDM mass (small-k limit preserved)
- Suppression of T_X(k) relative to T_CDM(k) grows monotonically at high k for every m_X
- Suppression is ordered by mass: smaller m_X gives stronger high-k suppression, consistent with fewer low-mass haloes than CDM
- Transfer function is interpolated with linear splines on the log-space function and covers the hmf k-range ~1e-8 to 2000 h/Mpc
- Resulting power spectrum normalises to the input sigma_8 before growth-factor application

## Limitations

- joezuntz/pycamb is out of date and no longer works with recent CAMB versions; CAMB now ships its own Python bindings
- pycamb was built for Python 2.5-2.7 and requires CAMB compiled with the same compiler/bitness as Python (e.g. gcc/gfortran)
- Any supplied transfer-function file must be CAMB-produced or in CAMB format; for ranges beyond the file, use extrapolate_with_eh=True (Eisenstein & Hu) at higher k
- WDM defaults fix ν = 1.2 and g_X = 1.5 (single-species), leaving m_X as the only free parameter, so non-default degeneracy assumptions need manual changes

## Evidence

- [intro] CAMB computes transfer functions used by hmf: "We use the public Code for Anisotropies in the Microwave Background (CAMB) (Lewis et al., 2000) to compute our transfer functions."
- [methods] Transfer-function input must be CAMB format: "If a filename is provided, the relevant file will be read and used as the transfer function. This file must be either produced by CAMB or in CAMB format."
- [methods] Log-space spline interpolation of T(k): "Interpolation of the transfer function is performed with linear splines on the function in log-space"
- [methods] WDM correction applied to power spectrum: "Find T WDM pkq Ź Eq 14 Ppkq Ð Ppkq ˚ T 2 WDM"
- [readme] pycamb transfers routine signature/output: [evidence span withheld — anchor status: not_found]
- [readme] pycamb deprecated for recent CAMB: [evidence span withheld — anchor status: not_found]
