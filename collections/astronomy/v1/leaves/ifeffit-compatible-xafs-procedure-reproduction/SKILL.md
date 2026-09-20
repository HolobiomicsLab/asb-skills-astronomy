---
name: ifeffit-compatible-xafs-procedure-reproduction
description: Use when processing raw XAS/XAFS spectra (e.g. ASCII energy/mu files
  like fe.060) to produce normalized mu(E) and background-subtracted chi(k), and when
  IFEFFIT-equivalent procedure semantics matter.
when_to_use_negative:
- The input is already a processed Athena project or chi(k) array and only downstream
  fitting/FT steps are needed — skip the autobk stage and read the group with read_athena
  instead.
- The task is RIXS processing — the paper notes Larch currently provides no GUI for
  RIXS, so this XAFS-processing skill does not cover it.
- Non-XAFS spectroscopy (XRF spectra, XRF/XRD mapping images) that requires Larch's
  XRF map viewer or XRD tools rather than the autobk/xftf chain.
edam_operation: http://edamontology.org/operation_0334
tools:
- name: Larch (xraylarch)
  role: Provides autobk, pre_edge, xftf, xftr, feffit_transform, read_ascii, and read_athena
    used for the IFEFFIT-compatible XAFS processing chain
  repo: https://github.com/xraypy/xraylarch
- name: Python
  role: Language in which Larch is written and in which processing scripts are authored
  repo: https://github.com/xraypy/xraylarch
- name: ifeffit
  role: Legacy XAFS package whose core procedures (normalization, background subtraction,
    Fourier transforms, fitting) Larch preserves; reference for parameter semantics
- name: feff
  role: Theory engine whose paths (via feffrunner, feffpath, ff2chi) feed the fitting
    of experimental spectra after background subtraction
  repo: https://github.com/xraypy/xraylarch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:9580b391765b19884238a504abbe21c7bcbed8b14420580cafc23ef5dfcba97c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xraypy/xraylarch
  tools:
  - Larch (xraylarch)
  - Python
  - feff
  - ifeffit
  - larch
  edam_operation: http://edamontology.org/operation_0334
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ifeffit-compatible-xafs-procedure-reproduction

## Summary

Reproduces the IFEFFIT-preserved XAFS analysis chain in Larch — pre-edge/normalization, autobk background subtraction to chi(k), and Fourier transforms — using documented parameters such as rbkg=1.0, kweight=2, and Hanning/kaiser FT windows. Larch deliberately keeps the core XAFS procedures from IFEFFIT with few algorithmic changes, so results are comparable to the legacy package.

## When to use

Use when processing raw XAS/XAFS spectra (e.g. ASCII energy/mu files like fe.060) to produce normalized mu(E) and background-subtracted chi(k), and when IFEFFIT-equivalent procedure semantics matter.

## When NOT to use

- The input is already a processed Athena project or chi(k) array and only downstream fitting/FT steps are needed — skip the autobk stage and read the group with read_athena instead.
- The task is RIXS processing — the paper notes Larch currently provides no GUI for RIXS, so this XAFS-processing skill does not cover it.
- Non-XAFS spectroscopy (XRF spectra, XRF/XRD mapping images) that requires Larch's XRF map viewer or XRD tools rather than the autobk/xftf chain.

## Inputs

- ASCII XAFS data file (energy, mu arrays) such as fe.060 from the xafsdata folder
- Athena project files (e.g. fe_athena.prj with Fe2O3/Fe3C/FeO data, cyanobacteria.prj) via read_athena
- Feff path outputs (feff6/feff8) for fitting experimental spectra to a sum of Feff paths

## Outputs

- Data group containing chi(k), spline background, e0, edge_step from autobk
- CSV export of chi(k) array and autobk fit parameters (e.g. fe060_autobk.csv)
- chi(R) Fourier transforms and R-space arrays from xftf/xftr with chosen window parameters

## How to apply

Load the raw spectrum with Larch's read_ascii (e.g. '../xafsdata/fe.060') into a data group, then run pre_edge to obtain e0 and edge_step (the paper reports e0=8980.5 eV, edge_step=2.729 for a Cu foil reference). Perform background subtraction with larch.xafs.autobk on the energy and mu arrays using rbkg=1.0 and kweight=2, optionally clamp_hi (0, 50, 200 for sensitivity testing), which stores chi(k), the spline background, e0, and edge_step in the group. Export the chi(k) array and fit parameters (e.g. to fe060_autobk.csv). For EXAFS analysis, apply xftf with kweight=2, kmin=3, kmax=13, window='hanning' and dk=1 (or dk=4-5 for window comparisons; gaussian/parzen/welch/kaiser are available), and use xftr (rmin=1, rmax=3.3, dr=0.2) for back-transforms. The rationale is that Larch preserves IFEFFIT's core procedures — normalization, background subtraction, Fourier transforms, XANES fitting, and fitting to Feff paths — while adding Python extensibility.

## Related tools

- **Larch (xraylarch)** (Provides autobk, pre_edge, xftf, xftr, feffit_transform, read_ascii, and read_athena used for the IFEFFIT-compatible XAFS processing chain) — https://github.com/xraypy/xraylarch
- **Python** (Language in which Larch is written and in which processing scripts are authored) — https://github.com/xraypy/xraylarch
- **ifeffit** (Legacy XAFS package whose core procedures (normalization, background subtraction, Fourier transforms, fitting) Larch preserves; reference for parameter semantics)
- **feff** (Theory engine whose paths (via feffrunner, feffpath, ff2chi) feed the fitting of experimental spectra after background subtraction) — https://github.com/xraypy/xraylarch

## Examples

```
from larch.xafs import autobk; data = read_ascii('../xafsdata/fe.060'); autobk(data.energy, data.mu, rbkg=1.0, kweight=2, group=data)
```

## Evaluation signals

- Output CSV (e.g. fe060_autobk.csv) exists and the chi(k) array is populated with finite values
- The data group exposes autobk fit parameters e0, edge_step, and the spline background (reference Cu foil: e0=8980.5 eV, edge_step=2.729)
- autobk was invoked with the documented settings rbkg=1.0 and kweight=2; clamp_hi sensitivity (0/50/200) reported if varied
- xftf called with explicit k-range and window (e.g. kweight=2, kmin=3, kmax=13, window='hanning', dk=1) rather than defaults
- Results are consistent with IFEFFIT-preserved procedures — few algorithmic changes are expected between Larch and IFEFFIT for normalization, background subtraction, and Fourier transforms

## Limitations

- Background subtraction quality is sensitive to clamp_hi: the article demonstrates autobk sensitivity tests with clamp_hi=0, 50, 200
- FT results depend on window and dk choice: hanning with dk=1 vs dk=5 differ, and window comparison (hanning, parzen, welch, kaiser, gaussian) must be controlled with shared ftopts
- Larch requires Python 2.7/3.5/3.6 plus several required Python packages, and system-Python installs may need super-user privileges
- Larch does not provide a GUI for RIXS processing, and Larix GUI coverage should not be assumed for every related measurement mode

## Evidence

- [abstract] the initial release of larch preserves the core XAFS analysis procedures such as normalization, background subtraction, Fourier transforms, fitting of XANES spectra, and fitting of experimental: "the initial release of larch preserves the core XAFS analysis procedures such as normalization, background subtraction, Fourier transforms, fitting of XANES spectra, and fitting of experimental"
- [results] data = read_ascii('../xafsdata/fe.060'): "data = read_ascii('../xafsdata/fe.060')"
- [results] autobk(dat.energy, dat.mu, rbkg=1.0, group=dat, clamp_hi=0): "autobk(dat.energy, dat.mu, rbkg=1.0, group=dat, clamp_hi=0)"
- [results] from larch.xafs import autobk, xftf, feffrunner, feffpath, ff2chi: [evidence span withheld — anchor status: not_found]
