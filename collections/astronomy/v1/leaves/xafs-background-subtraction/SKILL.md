---
name: xafs-background-subtraction
description: Use when a raw X-ray absorption spectrum (e.g. energy and mu arrays from
  an ASCII XAFS file such as fe.060) must be background-subtracted to yield chi(k)
  before Fourier transform or Feff fitting.
when_to_use_negative:
- Input is already a chi(k) array or an Athena project (fe_athena.prj) — background
  subtraction via autobk on mu(E) is not needed
- The next step is XANES normalization or pre-edge peak fitting where only pre_edge/normalized
  mu(E) is required, not EXAFS background removal
- Spectral data are not X-ray absorption spectra (e.g. XRF maps or RIXS planes), which
  Larch handles with other tools
edam_operation: http://edamontology.org/operation_0321
tools:
- name: Larch (xraylarch)
  role: Provides the autobk function (larch.xafs.autobk) and read_ascii loader used
    to background-subtract mu(E) into chi(k)
  repo: github.com/xraypy/xraylarch
- name: Python
  role: Runtime environment in which Larch's autobk workflow is scripted and results
    exported to CSV
  repo: github.com/xraypy/xraylarch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:526be059a7eb3caef13a7f17fecf0126e533d48654beab8ee8c95d1013eb1dbb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Larch (xraylarch)
  - Python
  - larch
  edam_operation: http://edamontology.org/operation_0321
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# xafs-background-subtraction

## Summary

Performs the XAFS background-subtraction step of Larch's processing workflow using the autobk function to remove the smooth pre-edge/post-edge background from mu(E) and produce chi(k), preserving the procedures inherited from IFEFFIT.

## When to use

Use when a raw X-ray absorption spectrum (e.g. energy and mu arrays from an ASCII XAFS file such as fe.060) must be background-subtracted to yield chi(k) before Fourier transform or Feff fitting.

## When NOT to use

- Input is already a chi(k) array or an Athena project (fe_athena.prj) — background subtraction via autobk on mu(E) is not needed
- The next step is XANES normalization or pre-edge peak fitting where only pre_edge/normalized mu(E) is required, not EXAFS background removal
- Spectral data are not X-ray absorption spectra (e.g. XRF maps or RIXS planes), which Larch handles with other tools

## Inputs

- Raw XAFS spectrum as energy and mu arrays (e.g. ASCII file fe.060 read via Larch read_ascii)
- Optional autobk settings: rbkg (e.g. 1.0), kweight (e.g. 2), clamp_hi (e.g. 0, 50, 200)

## Outputs

- chi(k) array stored in the Larch data group
- autobk fit parameters in the data group (e0, edge_step, spline background)
- Derived CSV of chi(k) and fit parameters (e.g. fe060_autobk.csv)

## How to apply

Load the raw XAFS spectrum with Larch's read_ascii (e.g. data = read_ascii('../xafsdata/fe.060')), giving a data group with energy and mu arrays. Call larch.xafs.autobk on the energy and mu arrays with the documented setting rbkg=1.0 (optionally kweight=2 and clamp_hi, e.g. clamp_hi=0, 50, or 200 for sensitivity tests), storing results in the same data group. The spline background fit also yields pre-edge/edge parameters such as e0 and edge_step in the group; extract the chi(k) array plus these fit parameters for downstream steps (xftf with kweight=2, kmin=3, kmax=13, hanning window). Rationale: Larch's initial release preserves the core XAFS analysis procedures from IFEFFIT with few algorithmic changes, so autobk reproduces the established background-subtraction behavior.

## Related tools

- **Larch (xraylarch)** (Provides the autobk function (larch.xafs.autobk) and read_ascii loader used to background-subtract mu(E) into chi(k)) — github.com/xraypy/xraylarch
- **Python** (Runtime environment in which Larch's autobk workflow is scripted and results exported to CSV) — github.com/xraypy/xraylarch

## Examples

```
from larch.xafs import autobk; data = read_ascii('../xafsdata/fe.060'); autobk(data.energy, data.mu, rbkg=1.0, group=data)
```

## Evaluation signals

- Output CSV (e.g. fe060_autobk.csv) exists and the chi(k) array is populated with finite (non-NaN) values
- Fit parameters e0 and edge_step are present and physically sensible in the data group (cf. Cu foil reference: e0=8980.5 eV, edge_step=2.729)
- Spline background tracks mu(E) outside the EXAFS region; rbkg=1.0 is recorded as the documented setting
- Varying clamp_hi (0, 50, 200) produces the expected small changes in chi(k) without instability, matching the documented sensitivity test
- Downstream xftf with kweight=2, kmin=3, kmax=13, window='hanning' yields a well-formed chi(R) transform from the autobk output

## Limitations

- Larch's initial release preserves IFEFFIT background-subtraction procedures with few algorithmic changes, so results mirror IFEFFIT rather than new algorithms
- The clamp_hi parameter influences how the spline background is clamped at high k; unsuitable values can distort chi(k), as shown by the 0/50/200 sensitivity test
- Larch does not provide a GUI for RIXS processing, and GUI coverage is uneven across spectroscopy modes

## Evidence

- [results] from larch.xafs import autobk, xftf, feffrunner, feffpath, ff2chi: [evidence span withheld — anchor status: not_found]
