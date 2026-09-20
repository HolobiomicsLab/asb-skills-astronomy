---
name: wdm-vs-cdm-suppression-quantification
description: Use when the analysis requires comparing predicted halo abundance (dndlnm/ngtm)
  between a CDM baseline and WDM models parameterised by particle mass in keV, or
  quantifying low-mass halo suppression from P(k) * T_WDM^2.
when_to_use_negative:
- When the input is an empirically measured HMF from an N-body simulation to be checked
  against theory — HMFcalc/hmf computes analytic fitting-function HMFs, not simulation
  post-processing
- When the dark matter model is beyond CDM/WDM (e.g. interacting or non-standard dark
  energy) — broadening the range of dark matter models is explicitly listed as future
  work in the paper
- When the only change needed is redshift — the cascading update path is optimised
  for that and a full re-comparison workflow is unnecessary
edam_operation: http://edamontology.org/operation_0244
tools:
- name: hmf
  role: 'Python engine that computes the HMF: MassFunction class, update() cascading
    parameter updates, dndlnm/ngtm outputs, WDM power-spectrum correction and growth-factor
    application'
  repo: https://github.com/halomod/hmf
- name: CAMB
  role: Computes the linear matter transfer function T(k) consumed by hmf for the
    power spectrum and mass variance
  repo: https://github.com/joezuntz/pycamb
- name: matplotlib
  role: Renders the four HMF curves (CDM plus three WDM models) as a PNG figure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:c4e92676e767c82e78392e3e1bbbb5cb680ebbfd75c6bd00b7aa885bdff68790
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - Python
  - hmf
  - matplotlib
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# wdm-vs-cdm-suppression-quantification

## Summary

Quantify how Warm Dark Matter transfer-function filtering (Bode et al. 2001) suppresses the halo mass function relative to a fiducial CDM baseline, using the hmf Python package with CAMB transfer functions, for m_X = 0.5, 1.0, 2.0 keV (reproducing the paper's Figure 4 workflow).

## When to use

Use when the analysis requires comparing predicted halo abundance (dndlnm/ngtm) between a CDM baseline and WDM models parameterised by particle mass in keV, or quantifying low-mass halo suppression from P(k) * T_WDM^2.

## When NOT to use

- When the input is an empirically measured HMF from an N-body simulation to be checked against theory — HMFcalc/hmf computes analytic fitting-function HMFs, not simulation post-processing
- When the dark matter model is beyond CDM/WDM (e.g. interacting or non-standard dark energy) — broadening the range of dark matter models is explicitly listed as future work in the paper
- When the only change needed is redshift — the cascading update path is optimised for that and a full re-comparison workflow is unnecessary

## Inputs

- Cosmological parameter set (hmf defaults; Omega_c, Omega_Lambda, sigma8, spectral index n)
- WDM particle masses m_X in keV/c^2 (0.5, 1.0, 2.0) applied via update(wdm_mass=...)
- Linear matter transfer function T(k) from CAMB (or a file in CAMB format)
- Logarithmic halo mass grid at 0.05 dex bin width

## Outputs

- CDM and WDM differential HMF curves dndlnm(M) and cumulative HMF curves ngtm(M)
- Suppression ratios dndlnm_WDM/dndlnm_CDM as a function of halo mass for each m_X
- wdm_hmf_suppression.csv containing the four curves and their ratios
- matplotlib PNG figure rendering the four HMF curves

## How to apply

Instantiate the hmf MassFunction class with its default cosmology so CAMB supplies the transfer function (linear-spline log-space interpolation, sigma8 normalisation) and compute baseline CDM dndlnm and ngtm on the default logarithmic mass grid at 0.05 dex bin width. For each WDM particle mass (0.5, 1.0, 2.0 keV), call the update() method so the cascading dependency tracker recomputes only affected quantities, applying the Bode et al. (2001) suppression factor as P(k) ← P(k) * T_WDM^2 (Eq 14). Apply the linear growth factor D(z) at z=0 to the power spectrum and mass variance (P(k) ← P0(k) D^2(z), sigma(M,z) ← sigma0(M) D(z)) before evaluating the fitting function f(sigma) and the HMF. For each m_X compute the suppression ratio dndlnm_WDM/dndlnm_CDM versus halo mass, tabulate the four curves, and write them to CSV alongside a matplotlib PNG of the HMF curves. The rationale is that the HMF is sensitive to the nature of the dark matter: viable WDM models predict fewer haloes at low masses than the CDM M^-1.8 rise.

## Related tools

- **hmf** (Python engine that computes the HMF: MassFunction class, update() cascading parameter updates, dndlnm/ngtm outputs, WDM power-spectrum correction and growth-factor application) — https://github.com/halomod/hmf
- **CAMB** (Computes the linear matter transfer function T(k) consumed by hmf for the power spectrum and mass variance) — https://github.com/joezuntz/pycamb
- **matplotlib** (Renders the four HMF curves (CDM plus three WDM models) as a PNG figure)

## Examples

```
from hmf import MassFunction; mf_cdm = MassFunction(); mf_wdm = MassFunction(wdm_mass=0.5); r = mf_wdm.dndlnm / mf_cdm.dndlnm
```

## Evaluation signals

- The CDM-normalised suppression ratio dndlnm_CDM/dndlnm_CDM must equal unity across the whole mass grid
- Low-mass suppression must increase monotonically as m_X decreases: 0.5 keV most suppressed, 2.0 keV least, consistent with the Bode et al. (2001) T_WDM(k) filtering
- The CDM baseline HMF must rise toward low mass at roughly the expected M^-1.8 slope
- CSV must contain all four HMF curves plus suppression ratios tabulated against the same 0.05 dex mass grid
- All curves must be evaluated after growth-factor application (P(k) ← P0(k) D^2(z), sigma ← sigma0 D(z)), not on the z=0 raw sigma8-normalised spectrum

## Limitations

- pycamb (joezuntz/pycamb) is outdated and no longer works with recent CAMB versions; modern CAMB ships its own Python bindings, so transfer functions should be obtained accordingly
- Legacy pycamb targets Python 2.5–2.7 (not Python 3) and requires CAMB compiled with the same compiler set as Python
- WDM handling is limited to the Bode et al. (2001) suppression factor applied as P(k) * T_WDM^2; HMF results also depend on the choice of f(sigma) fitting function
- No changelog found for the source repository, so version-to-version behavioural changes in the WDM or growth-factor implementations (e.g. the growth-factor PR referenced in the discussion) cannot be tracked from the repo alone

## Evidence

- [intro] CDM predicts the halo abundance rises with decreasing mass approximately as M^-1.8, while viable WDM models predict fewer haloes at low masses.: [evidence span withheld — anchor status: not_found]
- [methods] update() cascading update mechanism: [evidence span withheld — anchor status: not_found]
- [methods] WDM power-spectrum correction (Eq 14): [evidence span withheld — anchor status: not_found]
- [readme] pycamb is deprecated in favour of modern CAMB Python bindings: [evidence span withheld — anchor status: not_found]
