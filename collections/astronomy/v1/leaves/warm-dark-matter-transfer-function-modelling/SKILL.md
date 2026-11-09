---
name: warm-dark-matter-transfer-function-modelling
description: Use when the goal is to quantify how warm dark matter particle mass (e.g.
  m_X = 0.5–2 keV) suppresses low-mass halo abundance in the HMF relative to a CDM
  baseline power spectrum.
when_to_use_negative:
- When the task requires non-linear halo modelling (HOD, halo-model power spectra)
  rather than the linear-theory HMF comparison hmf/HMFcalc is designed for.
- When a WDM model other than the Bode et al. (2001) thermal-relic T_WDM(k) filter
  is required, or the question concerns self-interacting/decaying dark matter beyond
  the article's scope.
- 'When pycamb is required: the joezuntz/pycamb README states it is outdated and no
  longer works with recent CAMB versions, which now ship their own Python bindings.'
edam_operation: http://edamontology.org/operation_2426
tools:
- name: hmf
  role: Python engine used to instantiate MassFunction, update WDM parameters via
    update(), and compute dndlnm/ngtm on a 0.05 dex mass grid
  repo: https://github.com/halomod/hmf
- name: CAMB
  role: Computes the transfer functions underlying the CDM power spectrum, normalised
    by sigma8 and interpolated with linear splines in log-space
  repo: https://github.com/joezuntz/pycamb
- name: Python
  role: Implementation language of hmf and the WDM suppression workflow, including
    the matplotlib figure rendering
- name: HMFcalc
  role: Web frontend to hmf usable as a standard against which to check the computed
    HMFs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:3b67f7a3ae31670e44d762559c840b20fc4cf06d97b421014f90ce4e3a7ed807
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - HMFcalc
  - Python
  - hmf
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# warm-dark-matter-transfer-function-modelling

## Summary

Reproduce the CDM vs WDM halo mass function comparison of Murray et al. (2013) Figure 4 by applying the Bode et al. (2001) WDM suppression factor T_WDM(k) to a CAMB-derived transfer function inside the hmf MassFunction framework, for m_X = 0.5, 1.0 and 2.0 keV/c^2, and quantifying dndlnm suppression relative to a fiducial CDM baseline.

## When to use

Use when the goal is to quantify how warm dark matter particle mass (e.g. m_X = 0.5–2 keV) suppresses low-mass halo abundance in the HMF relative to a CDM baseline power spectrum.

## When NOT to use

- When the task requires non-linear halo modelling (HOD, halo-model power spectra) rather than the linear-theory HMF comparison hmf/HMFcalc is designed for.
- When a WDM model other than the Bode et al. (2001) thermal-relic T_WDM(k) filter is required, or the question concerns self-interacting/decaying dark matter beyond the article's scope.
- When pycamb is required: the joezuntz/pycamb README states it is outdated and no longer works with recent CAMB versions, which now ship their own Python bindings.

## Inputs

- hmf MassFunction object with default cosmology (CAMB transfer function, sigma8 normalisation, 0.05 dex logarithmic mass grid)
- WDM particle masses m_X = 0.5, 1.0, 2.0 keV/c^2
- Bode et al. (2001) WDM transfer-function filter T_WDM(k) (Eq 14)

## Outputs

- Four HMF curves (CDM plus three WDM models) as dndlnm and ngtm vs halo mass
- Suppression ratios dndlnm_WDM/dndlnm_CDM as a function of halo mass
- wdm_hmf_suppression.csv containing curves and ratios
- matplotlib PNG figure rendering the four HMF curves

## How to apply

Instantiate the hmf MassFunction class with its default cosmology so the transfer function is computed via CAMB (pycamb), interpolated with linear splines in log-space and normalised by sigma8. Compute the baseline CDM dndlnm and ngtm on the default logarithmic mass grid (0.05 dex bin width). Then call update() three times with WDM particle masses m_X = 0.5, 1.0 and 2.0 keV, which multiplies P(k) by T_WDM^2 (Bode et al. 2001, Eq 14). Apply the linear growth factor d(z) at z=0 to P(k) and the mass variance before evaluating the fitting function f(sigma) and the HMF, then compute the suppression ratio dndlnm_WDM/dndlnm_CDM per halo mass.

## Related tools

- **hmf** (Python engine used to instantiate MassFunction, update WDM parameters via update(), and compute dndlnm/ngtm on a 0.05 dex mass grid) — https://github.com/halomod/hmf
- **CAMB** (Computes the transfer functions underlying the CDM power spectrum, normalised by sigma8 and interpolated with linear splines in log-space) — https://github.com/joezuntz/pycamb
- **Python** (Implementation language of hmf and the WDM suppression workflow, including the matplotlib figure rendering)
- **HMFcalc** (Web frontend to hmf usable as a standard against which to check the computed HMFs)

## Examples

```
from hmf import MassFunction; mf = MassFunction(); mf.update(wdm_mass=1.0); mf.dndlnm; mf.ngtm
```

## Evaluation signals

- The CDM-normalised ratio dndlnm_WDM/dndlnm_CDM must equal unity for the baseline model.
- Low-mass suppression must increase monotonically as m_X decreases: 0.5 keV suppressed most, 2.0 keV least.
- Toward low halo mass the CDM HMF should rise approximately as M^-1.8, consistent with the article's stated CDM expectation.
- Suppression ratios must approach unity at high halo masses for all three WDM models, since T_WDM(k) only filters small-scale power.
- Output files wdm_hmf_suppression.csv and the PNG figure must contain all four curves (CDM + 3 WDM) on the shared mass grid.

## Limitations

- pycamb is out of date and no longer works with recent versions of CAMB; the README advises using CAMB's own Python bindings instead.
- The approach relies on the Bode et al. (2001) thermal-relic WDM transfer-function filter; other WDM scenarios are outside the article's workflow.
- Only redshift z=0 growth is applied here; the article notes HMF dependence on cosmological parameters and redshift via the growth factor d(z).

## Evidence

- [methods] Cascading update method: [evidence span withheld — anchor status: not_found]
- [intro] CDM vs WDM HMF expectation: [evidence span withheld — anchor status: not_found]
