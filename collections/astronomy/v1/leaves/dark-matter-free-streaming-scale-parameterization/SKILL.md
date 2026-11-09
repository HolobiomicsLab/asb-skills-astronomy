---
name: dark-matter-free-streaming-scale-parameterization
description: 'Use when computing halo mass functions under a WDM model: a CDM linear
  transfer function is available and the small-scale power must be suppressed via
  a free-streaming scale set by m_X, Omega_X, h and g_X.'
when_to_use_negative:
- The model is plain CDM (no WDM component selected) — hmf only applies the T_WDM
  correction when a WDM model is chosen.
- Recent CAMB versions are the transfer-function source — pycamb is deprecated ('out
  of data and no longer works with recent versions of camb'); use CAMB's built-in
  Python bindings instead.
- You need a non-single-species WDM model or a custom g_X/nu — HMFcalc's default fixes
  nu and g_X at fiducial values (g_X = 1.5), leaving only m_X free.
tools:
- name: hmf
  role: Engine providing the WDM correction module (in the style of hmf.alternatives.wdm)
    that applies P(k) <- P(k) * T^2_WDM to the CAMB transfer function before sigma_8
    normalisation.
  repo: https://github.com/halomod/hmf
- name: HMFcalc
  role: Web frontend that exposes the WDM power-spectrum filter to users and labels
    output legends by parameter combination.
  repo: http://hmf.icrar.org
- name: pycamb
  role: Generates the CDM linear transfer functions (k, T_i(k,z), sigma8) that the
    WDM filter T_X(k) is multiplied onto.
  repo: https://github.com/joezuntz/pycamb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:31683940067a6d979969f259738cee7ab28fcc07cd435adddc1e4d4d9c1f4d62
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - HMFcalc
  - Python
  - hmf
  - pycamb
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# dark-matter-free-streaming-scale-parameterization

## Summary

Parameterizes the free-streaming suppression of small-scale power in warm dark matter (WDM) models by applying the Bode et al. (2001) transfer function T_X(k) to a CDM linear power spectrum, as implemented in HMFcalc/hmf.

## When to use

Use when computing halo mass functions under a WDM model: a CDM linear transfer function is available and the small-scale power must be suppressed via a free-streaming scale set by m_X, Omega_X, h and g_X.

## When NOT to use

- The model is plain CDM (no WDM component selected) — hmf only applies the T_WDM correction when a WDM model is chosen.
- Recent CAMB versions are the transfer-function source — pycamb is deprecated ('out of data and no longer works with recent versions of camb'); use CAMB's built-in Python bindings instead.
- You need a non-single-species WDM model or a custom g_X/nu — HMFcalc's default fixes nu and g_X at fiducial values (g_X = 1.5), leaving only m_X free.

## Inputs

- CDM linear transfer function T(k) in CAMB format (from CAMB or a CAMB-produced file)
- Cosmological parameters Omega_X (WDM density), h, sigma_8
- WDM particle mass m_X (keV) and degeneracy g_X (default 1.5)
- Shape parameter nu (default 1.2)

## Outputs

- WDM transfer function T_X(k) tabulated on the shared k-grid per m_X (e.g. wdm_transfer_functions.csv)
- WDM-filtered power spectrum P(k) = P_CDM(k) * T^2_WDM(k)
- Derived WDM halo mass function (dndlnm, ngtm) showing suppressed low-mass halo abundance

## How to apply

Obtain the CDM linear transfer function T(k) from CAMB (via pycamb.transfers) over the hmf default k-range, then implement the WDM filter as T_X(k) = T_CDM(k) * [1 + (alpha*k)^(2*1.2)]^(-1), where alpha follows Eqs 14-15 of Bode et al. (2001) in terms of Omega_X, h, m_X and g_X. HMFcalc fixes the shape parameter nu = 1.2 and, by default, g_X = 1.5 (single-species model), so m_X is the only free parameter. Per the hmf pipeline, the WDM correction multiplies the power spectrum as P(k) <- P(k) * T^2_WDM (Eq 14) before sigma_8 normalisation and application of the linear growth factor D(z). Evaluate T_X(k) on a shared k-grid for the desired m_X values (e.g. 0.5, 1.0, 2.0 keV) and tabulate results with columns labelled by parameter combination, as HMFcalc labels legends. Rationale: viable WDM models suppress the HMF at low masses relative to CDM, and the Bode et al. filter captures this free-streaming cutoff analytically without rerunning a Boltzmann code.

## Related tools

- **hmf** (Engine providing the WDM correction module (in the style of hmf.alternatives.wdm) that applies P(k) <- P(k) * T^2_WDM to the CAMB transfer function before sigma_8 normalisation.) — https://github.com/halomod/hmf
- **HMFcalc** (Web frontend that exposes the WDM power-spectrum filter to users and labels output legends by parameter combination.) — http://hmf.icrar.org
- **pycamb** (Generates the CDM linear transfer functions (k, T_i(k,z), sigma8) that the WDM filter T_X(k) is multiplied onto.) — https://github.com/joezuntz/pycamb

## Examples

```
T_WDM = T_CDM * (1 + (alpha*k)**(2*1.2))**-1  # apply Bode et al. (2001) filter, then P(k) *= T_WDM**2
```

## Evaluation signals

- T_X(k) -> 1 at small k (no suppression on large scales).
- Suppression grows monotonically at high k for every m_X.
- Suppression is ordered by mass: smaller m_X gives stronger small-scale suppression at fixed g_X = 1.5.
- The power spectrum is modified as P(k) * T^2_WDM (Eq 14), not by T_WDM to the first power.
- Resulting HMF shows fewer low-mass haloes than the CDM case, consistent with M ~ M^-1.8 CDM behaviour at the low-mass end.

## Limitations

- nu and g_X are fixed at fiducial values (nu = 1.2, g_X = 1.5) by default, so m_X is the only free parameter; non-fiducial species counts require overriding the defaults.
- pycamb is explicitly deprecated: 'Don't use this code... out of data and no longer works with recent versions of camb'; CAMB now ships its own Python bindings.
- If an external transfer-function file is supplied it must be CAMB-produced or in CAMB format; extrapolation to higher k beyond the CAMB range falls back to Eisenstein & Hu's fitting formula (extrapolate_with_eh=True).

## Evidence

- [methods] HMFcalc implements the WDM correction by adopting the Bode et al. (2001) transfer function T_X(k) = (1 + (αk)^{2ν})^{-5/ν}, with ν = 1.2: [evidence span withheld — anchor status: not_found]
- [intro] The standard Cold Dark Matter (CDM) model predicts an HMF in which the number of haloes increases with decreasing halo mass M approximately as M´1.8: [evidence span withheld — anchor status: not_found]
