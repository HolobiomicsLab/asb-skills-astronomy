---
name: xafs-fourier-transform-analysis
description: Use when a chi(k) array from background subtraction (autobk) must be
  converted to a chi(R) spectrum, or when comparing FT window choices (hanning vs
  parzen/welch/kaiser/gaussian, dk=1 vs 5) for EXAFS data.
when_to_use_negative:
- The input is raw mu(E) data not yet normalized or background-subtracted — run pre_edge/autobk
  first (Larch's pipeline is normalization, background subtraction, then Fourier transforms).
- The goal is back-transforming chi(R) to filtered chi(k) — use Larch's xftr() (e.g.,
  rmin=1, rmax=3.3, dr=0.2, hanning) instead of xftf.
- The task is EXAFS path fitting, where the transform is embedded in feffit_transform
  (kmin=3, kmax=12, kweight=2, dk=4, kaiser, rmin=1.0, rmax=3.0) rather than a standalone
  xftf call.
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3418
tools:
- name: larch
  role: Python package whose larch.xafs module provides xftf() and ftwindow() for
    the chi(k)→chi(R) transform
  repo: https://github.com/xraylarch
- name: Python
  role: language in which Larch is written and in which the xftf/ftwindow calls are
    scripted
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:48181a19434db01f3cc628c047821c460d229f08b263336cb1985bdd85fc1a29
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - larch
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# xafs-fourier-transform-analysis

## Summary

Fourier-transform stage of XAFS analysis in Larch: applying an FT window to autobk-derived chi(k) with xftf() to produce a chi(R) spectrum, and characterizing how window parameters (kmin, kmax, dk, kweight, window type) shape the result.

## When to use

Use when a chi(k) array from background subtraction (autobk) must be converted to a chi(R) spectrum, or when comparing FT window choices (hanning vs parzen/welch/kaiser/gaussian, dk=1 vs 5) for EXAFS data.

## When NOT to use

- The input is raw mu(E) data not yet normalized or background-subtracted — run pre_edge/autobk first (Larch's pipeline is normalization, background subtraction, then Fourier transforms).
- The goal is back-transforming chi(R) to filtered chi(k) — use Larch's xftr() (e.g., rmin=1, rmax=3.3, dr=0.2, hanning) instead of xftf.
- The task is EXAFS path fitting, where the transform is embedded in feffit_transform (kmin=3, kmax=12, kweight=2, dk=4, kaiser, rmin=1.0, rmax=3.0) rather than a standalone xftf call.

## Inputs

- chi(k) array produced by autobk background subtraction of a normalized XAFS spectrum (e.g., Fe foil data)
- k grid aligned with the chi(k) array
- FT window parameters: window type (hanning, parzen, welch, kaiser, gaussian), kmin, kmax, dk, kweight

## Outputs

- complex chi(R) spectrum (magnitude and real part) on an R grid
- applied FT window array over k (from ftwindow)
- Larch group exposing the FT window parameters actually used (window, kmin, kmax, dk, kweight)
- chir_re.csv containing the chi(R) array plus recorded window parameters

## How to apply

Load the chi(k) array produced by the autobk step, then optionally generate the window explicitly with Larch's ftwindow() (e.g., xmin=3, xmax=13, dx=1, window='hanning'). Run larch.xafs.xftf with the desired parameters — kweight=2, kmin=3, kmax=13, window='hanning', dk=1 is a canonical setting from the paper's example — which applies the FT window to chi(k) and computes the complex chi(R) spectrum. To probe sensitivity, repeat with different dk values (e.g., dk=5) or other window types (parzen, welch, kaiser, gaussian with dk=4) and compare the resulting chi(R) spectra. Larch preserves the core IFEFFIT XAFS procedures, including Fourier transforms, so results follow the same algorithmic conventions. Extract the window parameters from the returned group and write the chi(R) magnitude/real part alongside them to chir_re.csv for downstream validation.

## Related tools

- **larch** (Python package whose larch.xafs module provides xftf() and ftwindow() for the chi(k)→chi(R) transform) — https://github.com/xraylarch
- **Python** (language in which Larch is written and in which the xftf/ftwindow calls are scripted)

## Examples

```
from larch.xafs import xftf; xftf(dat, kweight=2, kmin=3, kmax=13, window='hanning', dk=1)
```

## Evaluation signals

- chir_re.csv exists and contains exactly one chi(R) value per R grid point.
- Window parameters extracted from the returned Larch group equal the requested values (window='hanning', kmin=3, kmax=13, dk=1, kweight=2).
- Varying dk (1 vs 5) or window type with the same k-range produces visibly different windowed chi(k) and chi(R) spectra, confirming the window is actually applied.
- The chi(R) spectrum is complex (magnitude and real part available), consistent with xftf's complex output convention inherited from IFEFFIT.

## Limitations

- The FT result depends strongly on window parameters (kmin, kmax, dk, kweight, window type); the paper demonstrates these choices (dk=1 vs dk=5; hanning/parzen/welch/kaiser/gaussian) rather than prescribing one universally correct setting.
- xftf requires chi(k) from a prior autobk step — it cannot operate on raw absorption data.
- The companion xftr back-transform uses a different parameterization (rmin/rmax/dr in R-space), so k-space window parameters do not transfer directly.

## Evidence

- [results] xftf importable from larch.xafs: [evidence span withheld — anchor status: not_found]
