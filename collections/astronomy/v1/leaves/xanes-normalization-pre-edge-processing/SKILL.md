---
name: xanes-normalization-pre-edge-processing
description: Use when a raw mu(E) XAS spectrum (e.g., ASCII Fe K-edge foil data like
  fe.060, or an Athena .prj project) must be edge-located and normalized before autobk
  background subtraction, EXAFS Fourier transforms, or XANES fitting.
when_to_use_negative:
- The spectrum is already normalized inside an Athena project file and you only need
  downstream steps (xftf/feffit), not re-normalization
- You are processing RIXS planes — Larch currently provides no GUI for RIXS processing
- You need an algorithmically new normalization method rather than the IFEFFIT-preserving
  procedures Larch retains with few changes
tools:
- name: Larch (xraylarch)
  role: Provides read_ascii, read_athena, pre_edge, and autobk used to load, normalize,
    and background-subtract XAFS spectra into data groups
  repo: https://github.com/xraypy/xraylarch
- name: Larix
  role: GUI application in Larch for XAFS processing and analysis, an alternative
    to scripted pre_edge/autobk use
  repo: https://github.com/xraypy/xraylarch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:c94692d1540f345aad3f2147756f5268ce4e35d2736fbc694453043c5a0b42a1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xraypy/xraylarch
  tools:
  - Larch (xraylarch)
  - Larix
  - larch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# xanes-normalization-pre-edge-processing

## Summary

Pre-edge/normalization processing of raw X-ray absorption spectra in Larch (xraylarch), which preserves the core XAFS normalization procedures from IFEFFIT. It establishes e0, edge_step, and the post-edge normalized mu(E) needed before autobk background subtraction or XANES fitting.

## When to use

Use when a raw mu(E) XAS spectrum (e.g., ASCII Fe K-edge foil data like fe.060, or an Athena .prj project) must be edge-located and normalized before autobk background subtraction, EXAFS Fourier transforms, or XANES fitting.

## When NOT to use

- The spectrum is already normalized inside an Athena project file and you only need downstream steps (xftf/feffit), not re-normalization
- You are processing RIXS planes — Larch currently provides no GUI for RIXS processing
- You need an algorithmically new normalization method rather than the IFEFFIT-preserving procedures Larch retains with few changes

## Inputs

- Raw ASCII XAFS spectrum (e.g., fe.060 from the xafsdata folder) with energy and mu arrays
- Athena project file (e.g., fe_athena.prj containing Fe2O3, Fe3C, FeO data) read with read_athena

## Outputs

- Larch data group populated with e0, edge_step, normalized mu, and (after autobk) chi(k) arrays
- Exported arrays/fit parameters, e.g., fe060_autobk.csv containing chi(k) and autobk fit parameters (e0, edge_step, spline background)

## How to apply

Load the raw spectrum with Larch's read_ascii (e.g., data = read_ascii('../xafsdata/fe.060')) into a data group. Run pre_edge on the group with the common normalization window settings used across groups in the Larch workflows: pre1=-150, pre2=-30, nnorm=1, norm1=150, norm2=850, which determines e0, edge_step, and the spline/normalization coefficients consistent with the IFEFFIT-preserved procedures. For downstream chi(k), follow with larch.xafs.autobk using rbkg=1.0 and kweight=2 (optionally clamp_hi values such as 0, 50, 200 for sensitivity testing) so the background subtraction inherits the pre_edge-derived e0 and edge_step. The rationale is that Larch's initial release preserves core XAFS procedures—normalization, background subtraction, Fourier transforms—with few algorithmic changes from IFEFFIT, while Python enables scripting of these steps over many groups.

## Related tools

- **Larch (xraylarch)** (Provides read_ascii, read_athena, pre_edge, and autobk used to load, normalize, and background-subtract XAFS spectra into data groups) — https://github.com/xraypy/xraylarch
- **Larix** (GUI application in Larch for XAFS processing and analysis, an alternative to scripted pre_edge/autobk use) — https://github.com/xraypy/xraylarch

## Examples

```
from larch.xafs import read_ascii, pre_edge, autobk; data = read_ascii('../xafsdata/fe.060'); pre_edge(data, pre1=-150, pre2=-30, nnorm=1, norm1=150, norm2=850); autobk(data, rbkg=1.0, kweight=2)
```

## Evaluation signals

- The data group's e0 and edge_step are populated with finite numeric values (reference Cu foil result: e0=8980.5 eV, edge_step=2.729)
- After autobk with rbkg=1.0, kweight=2, the chi(k) array is populated with finite values and the spline background is present
- The validation output file (e.g., fe060_autobk.csv) exists and contains both chi(k) and extracted fit parameters
- Normalization windows match the documented common settings: pre1=-150, pre2=-30, nnorm=1, norm1=150, norm2=850
- clamp_hi sensitivity tests (values 0, 50, 200) on autobk should be inspected to confirm the background is stable at the high-k end

## Limitations

- Larch's initial release preserves IFEFFIT's core XAFS procedures with few algorithmic changes, so results inherit IFEFFIT's normalization behavior rather than offering new algorithms
- Results depend on pre-edge/norm window choices (pre1, pre2, nnorm, norm1, norm2); the article applies a single common parameter set across PCA groups, which may not suit all edges
- autobk background quality is sensitive to clamp_hi settings (tested with 0, 50, 200), so chi(k) can vary with spline clamping choices

## Evidence
