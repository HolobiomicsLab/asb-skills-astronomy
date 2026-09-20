---
name: python-spectroscopy-data-export
description: Use when a Python (Larch) XAFS workflow has produced chi(k) from autobk
  background subtraction and you must compute and persist the chi(R) Fourier-transform
  spectrum plus its window parameters as a reusable CSV artifact.
when_to_use_negative:
- The goal is EXAFS path fitting rather than spectrum export — use feffit with a feffit_transform
  (kmin=3, kmax=12, kweight=2, dk=4, kaiser window, rmin=1.0, rmax=3.0) instead of
  a standalone xftf export.
- The input is raw mu(E) data not yet background-subtracted — xftf requires chi(k)
  from autobk as input, so run the autobk step first.
- A back-transform from R-space to k-space is needed — use larch's xftr (e.g. rmin=1,
  rmax=3.3, dr=0.2, window='hanning') rather than xftf.
tools:
- name: larch (xraylarch)
  role: Python library providing ftwindow and xftf to compute the Hanning-windowed
    Fourier transform of chi(k) into chi(R); also supplies autobk for the upstream
    chi(k) input
  repo: https://github.com/xraypy/xraylarch
- name: Python
  role: Language in which the xftf transform and the CSV export of chi(R) with window
    parameters are scripted
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:794afde9494fe6994a93a35f380f94eaa2f703ea41d93e0d4b1a9a73948cdd96
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xraypy/xraylarch
  tools:
  - Python
  - larch
  - larch (xraylarch)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# python-spectroscopy-data-export

## Summary

Exports derived spectroscopy products — here the chi(R) spectrum produced by Larch's Fourier-transform stage of XAFS analysis — from Python into a plain CSV file together with the processing parameters (FT window type, kmin, kmax, dk, kweight) needed to reproduce it.

## When to use

Use when a Python (Larch) XAFS workflow has produced chi(k) from autobk background subtraction and you must compute and persist the chi(R) Fourier-transform spectrum plus its window parameters as a reusable CSV artifact.

## When NOT to use

- The goal is EXAFS path fitting rather than spectrum export — use feffit with a feffit_transform (kmin=3, kmax=12, kweight=2, dk=4, kaiser window, rmin=1.0, rmax=3.0) instead of a standalone xftf export.
- The input is raw mu(E) data not yet background-subtracted — xftf requires chi(k) from autobk as input, so run the autobk step first.
- A back-transform from R-space to k-space is needed — use larch's xftr (e.g. rmin=1, rmax=3.3, dr=0.2, window='hanning') rather than xftf.

## Inputs

- chi(k) array produced by the autobk background-subtraction step for a target XAFS spectrum (e.g. Fe foil), with its k grid
- FT window settings: window type (e.g. 'hanning'), kmin=3, kmax=13, dk (e.g. 1 or 5), kweight=2

## Outputs

- chi(R) spectrum (magnitude/real part) with one value per R grid point, written to chir_re.csv
- Recorded FT window parameters (window='hanning', kmin=3, kmax=13, dk=1, kweight=2) exported alongside the spectrum

## How to apply

Load the autobk-derived chi(k) array for the target XAFS spectrum (e.g. the Fe foil), then build the FT window with larch's ftwindow function (e.g. xmin=3, xmax=13, dx=1, window='hanning') storing the window array alongside the k grid. Run larch.xafs.xftf on the chi(k) data with kweight=2, kmin=3, kmax=13, window='hanning', dk=1 to obtain the complex chi(R) spectrum; Larch preserves the core XAFS Fourier-transform procedures from IFEFFIT with few algorithmic changes, so these parameters map directly onto the familiar Athena/IFEFFIT conventions. Extract the window parameters actually recorded in the resulting Larch group (window type, kmin, kmax, dk, kweight) rather than trusting the call arguments alone, since these are the provenance metadata that accompany the exported data. Write the chi(R) magnitude/real-part spectrum with one value per R grid point, together with the window parameters, to a CSV file such as chir_re.csv. Larch is used here as a Python library — the same processing can also be driven interactively, in batch mode, or through the Larix GUI — but the export step is a scripted Python operation.

## Related tools

- **larch (xraylarch)** (Python library providing ftwindow and xftf to compute the Hanning-windowed Fourier transform of chi(k) into chi(R); also supplies autobk for the upstream chi(k) input) — https://github.com/xraypy/xraylarch
- **Python** (Language in which the xftf transform and the CSV export of chi(R) with window parameters are scripted)

## Examples

```
from larch.xafs import xftf; xftf(dat.chi, kweight=2, kmin=3, kmax=13, window='hanning', dk=1, group=dat)
```

## Evaluation signals

- chir_re.csv exists and contains a chi(R) array with exactly one value per R grid point
- Window parameters extracted from the resulting Larch group equal the requested values: window='hanning', kmin=3, kmax=13, dk=1, kweight=2
- The exported chi(R) reproduces when xftf is re-run with the recorded parameters, confirming the metadata are complete provenance
- Comparing dk=1 versus dk=5 exports shows the expected broadening/peak-shape differences of the chi(R) spectrum between window settings

## Limitations

- Larch does not provide a GUI for RIXS processing; related measurement modes are not all covered by GUI tools, so scripted export is required for some modes
- Window choice matters: the article exercises multiple window types (hanning, parzen, welch, kaiser, gaussian with dk=4) and dk values (1, 5), so an exported chi(R) is only valid for the specific window parameters recorded
- Larch preserves the core XAFS procedures 'with few algorithmic changes compared to IFEFFIT' — results should match legacy IFEFFIT analyses, so large deviations indicate a parameter or input error rather than a new algorithm

## Evidence

- [results] from larch.xafs import autobk, xftf, feffrunner, feffpath, ff2chi: [evidence span withheld — anchor status: not_found]
