---
name: halo-mass-function-computation
description: Use when a task requires calculating the differential (dndlnm) or cumulative
  (ngtm) halo mass function for a CDM or WDM cosmology, comparing fitting functions
  f(sigma), or quantifying WDM suppression of low-mass haloes relative to a CDM baseline.
when_to_use_negative:
- The analysis needs an independently measured HMF from N-body simulation halo catalogues
  rather than a fitting-function prediction on a smooth mass grid
- A transfer function outside CAMB's supported cosmologies is required and no CAMB-format
  file is available (hmf can only read files 'produced by CAMB or in CAMB format',
  or extrapolate with Eisenstein & Hu's fitting formula)
- Non-standard dark matter or dark energy models beyond the models supported by hmf/HMFcalc
  are needed (listed as future work in the paper)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3298
tools:
- name: hmf
  role: 'Python engine that computes the HMF: CAMB transfer functions, sigma_8 normalisation,
    fitting functions, cascading update() mechanism; instantiated as MassFunction'
  repo: https://github.com/halomod/hmf
- name: HMFcalc
  role: Web frontend to the hmf engine for interactive exploration of cosmological
    parameters, redshift and fitting function effects on the HMF
  repo: https://github.com/steven-murray/HMF
- name: CAMB
  role: Computes the transfer functions required for the linear power spectrum underlying
    the HMF calculation, accessed via pycamb
  repo: https://github.com/joezuntz/pycamb
- name: Python
  role: Implementation language for hmf/HMFcalc; used to drive MassFunction, call
    update(), and write CSV/matplotlib outputs
- name: pycamb
  role: Python bindings that let hmf call CAMB's transfer-function evaluation (pycamb.transfers
    returns k, T_i(k,z), sigma8(z))
  repo: https://github.com/joezuntz/pycamb
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:b004a6ef76770e883ca2e1794c42c112e6d741a6d8034b4ae83658668869fe26
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - HMFcalc
  - Python
  - hmf
  - pycamb
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3298
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# halo-mass-function-computation

## Summary

Compute the dark matter halo mass function (HMF) with the hmf Python package/HMFcalc, which obtains transfer functions from CAMB, normalises the power spectrum via sigma_8, applies WDM suppression factors where needed, and evaluates fitting functions like Tinker (2008) or Sheth-Tormen on a log-space mass grid.

## When to use

Use when a task requires calculating the differential (dndlnm) or cumulative (ngtm) halo mass function for a CDM or WDM cosmology, comparing fitting functions f(sigma), or quantifying WDM suppression of low-mass haloes relative to a CDM baseline.

## When NOT to use

- The analysis needs an independently measured HMF from N-body simulation halo catalogues rather than a fitting-function prediction on a smooth mass grid
- A transfer function outside CAMB's supported cosmologies is required and no CAMB-format file is available (hmf can only read files 'produced by CAMB or in CAMB format', or extrapolate with Eisenstein & Hu's fitting formula)
- Non-standard dark matter or dark energy models beyond the models supported by hmf/HMFcalc are needed (listed as future work in the paper)

## Inputs

- Cosmological parameter set (e.g. default PLANCK15; sigma_8 normalisation) supplied to hmf's MassFunction class
- Fitting-function string identifier (e.g. 'ST', 'Tinker', 'PS', 'Jenkins', 'Warren', 'Watson')
- Optional WDM particle mass m_X in keV (e.g. 0.5, 1.0, 2.0 keV) applying the Bode et al. (2001) transfer-function suppression
- Optional redshift z (default 0) for growth-factor rescaling
- Optionally a CAMB-produced transfer-function file in CAMB format, read in place of computing the transfer function

## Outputs

- Mass grid in Msun/h and differential mass function dn/dlnM (dndlnm) per halo mass bin
- Cumulative mass function n(>M) (ngtm) computed via linear spline interpolation and Romberg integration
- Suppression ratio curves dndlnm_WDM/dndlnm_CDM per WDM particle mass
- CSV tables (e.g. hmf_dndlnm.csv, wdm_hmf_suppression.csv) and matplotlib PNG figures of the HMF curves

## How to apply

Instantiate the hmf MassFunction class with default parameters (PLANCK15 cosmology, sigma_8 normalisation, top-hat window, 10^10-10^15 Msun/h mass grid at z=0); the transfer function is computed with CAMB (via pycamb), interpolated with linear splines in log-space and re-gridded between k-bounds in 4097 steps. The unnormalised power spectrum P0(k) = k^n T^2 is normalised by the mass variance sigma_8, then the linear growth factor d(z) is applied to P(k) and sigma(M,z) before evaluating the fitting function f(sigma) and the HMF. For WDM scenarios, call update() with the WDM particle mass (e.g. m_X = 0.5, 1.0, 2.0 keV) so the Bode et al. (2001) suppression T_WDM(k) is applied as P(k) * T_WDM^2 (Eq 14), then compute suppression ratios dndlnm_WDM/dndlnm_CDM. To switch fitting functions (PS, Jenkins, ST, Warren, Tinker, Watson, etc.), pass the string identifier to update() so the cascading dependency tracker deletes and recomputes only dependent quantities (power, sigma, n_eff, fsigma, dndlnm, ngtm), which reduces processing time by ~50% when each parameter is changed once and >94% when only redshift changes. Write mass grid (Msun/h) and dn/dlnM columns to CSV and optionally render curves with matplotlib.

## Related tools

- **hmf** (Python engine that computes the HMF: CAMB transfer functions, sigma_8 normalisation, fitting functions, cascading update() mechanism; instantiated as MassFunction) — https://github.com/halomod/hmf
- **HMFcalc** (Web frontend to the hmf engine for interactive exploration of cosmological parameters, redshift and fitting function effects on the HMF) — https://github.com/steven-murray/HMF
- **CAMB** (Computes the transfer functions required for the linear power spectrum underlying the HMF calculation, accessed via pycamb) — https://github.com/joezuntz/pycamb
- **Python** (Implementation language for hmf/HMFcalc; used to drive MassFunction, call update(), and write CSV/matplotlib outputs)
- **pycamb** (Python bindings that let hmf call CAMB's transfer-function evaluation (pycamb.transfers returns k, T_i(k,z), sigma8(z))) — https://github.com/joezuntz/pycamb

## Examples

```
from hmf import MassFunction; mf = MassFunction(); mf.update(hmf_model='Tinker'); import numpy as np; np.savetxt('hmf_dndlnm.csv', np.column_stack([mf.m, mf.dndlnm]), header='M[Msun/h],dndlnm')
```

## Evaluation signals

- CDM-normalised suppression ratio dndlnm_WDM/dndlnm_CDM equals unity for the baseline and low-mass suppression increases monotonically as m_X decreases (0.5 keV suppressed most, 2.0 keV least)
- Output table covers the full default 10^10-10^15 Msun/h mass span and dn/dlnM values fall in the expected range for the default cosmology
- Fitting-function comparison table contains one dndlnm column per fitting function plus 13 ratio columns, with order-unity ratios across most of the mass range and deviations concentrated at the high-mass end
- Cascading update() benchmarks reproduce ~50% time reduction when each parameter is changed once and >94% when only redshift changes
- Cross-check computed HMFs against genmf, MF_Code and the yt-project HMF calculator, with which hmf was validated for excellent consistency

## Limitations

- The standalone pycamb repository is deprecated: 'Don't use this code ... Since camb itself now ships with python bindings you should use those instead'
- Reading a transfer function from file requires the file to be 'either produced by CAMB or in CAMB format'; for CAMB's limited k-range one may need extrapolate_with_eh=True using Eisenstein & Hu's fitting formula at high k
- WDM suppression analysis is limited to the Bode et al. (2001) T_WDM(k) form; the paper lists broadening the range of dark matter and dark energy models as future work

## Evidence

- [intro] CDM vs WDM halo abundance expectation: [evidence span withheld — anchor status: not_found]
