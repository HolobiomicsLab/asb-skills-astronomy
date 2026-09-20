---
name: ascii-spectral-data-reading
description: Use when the input is a plain-text, column-delimited XAFS/XAS spectrum
  (e.g., fe.060 from an xafsdata folder) that must be ingested into a Larch group
  before pre_edge/autobk/xftf processing.
when_to_use_negative:
- Input is already a processed Athena project file (.prj) — use read_athena instead,
  as the article does with fe_athena.prj
- Input is a CIF/crystal structure file (e.g., from AMCSD or Crystallography Open
  Database) intended for Feff input generation, not a measured spectrum
- Input is a raw image or XRF map rather than a 1-D ASCII spectrum — Larch handles
  XRF/XRD images with different tools
edam_operation: http://edamontology.org/operation_0336
tools:
- name: Larch (xraylarch)
  role: Provides the read_ascii function used to parse ASCII XAFS files into a data
    group for the XAFS processing workflow
  repo: https://github.com/xraypy/xraylarch
- name: Python
  role: Execution environment; Larch is written in Python and used as a Python library
    for scripted XAFS processing
  repo: https://github.com/xraypy/xraylarch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:c7ad9ca7cfedb8bd83287ff25366517ddde32b29173bf97fb4538b4b21021438
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xraypy/xraylarch
  tools:
  - Larch (xraylarch)
  - Python
  - larch
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ascii-spectral-data-reading

## Summary

Load raw, column-based ASCII X-ray absorption spectra (e.g., the Fe K-edge foil file fe.060) into a Larch data group with read_ascii, giving energy and mu arrays ready for downstream XAFS steps such as pre-edge normalization and autobk background subtraction.

## When to use

Use when the input is a plain-text, column-delimited XAFS/XAS spectrum (e.g., fe.060 from an xafsdata folder) that must be ingested into a Larch group before pre_edge/autobk/xftf processing.

## When NOT to use

- Input is already a processed Athena project file (.prj) — use read_athena instead, as the article does with fe_athena.prj
- Input is a CIF/crystal structure file (e.g., from AMCSD or Crystallography Open Database) intended for Feff input generation, not a measured spectrum
- Input is a raw image or XRF map rather than a 1-D ASCII spectrum — Larch handles XRF/XRD images with different tools

## Inputs

- Column-based ASCII XAFS data file (e.g., fe.060 from the xafsdata folder) containing energy and mu columns

## Outputs

- Larch data group with parsed energy and mu arrays (ready for pre_edge, autobk, xftf)
- Downstream artifacts derived from the group, e.g., fe060_autobk.csv containing chi(k) and fit parameters (e0, edge_step, spline background)

## How to apply

Call Larch's read_ascii on the ASCII file path (e.g., '../xafsdata/fe.060') and keep the returned group as the working data object; the group exposes the energy and mu arrays used by later steps in the workflow. The article's results section shows exactly this pattern as the first line of the Fe K-edge processing example, preceding pre-edge normalization and autobk. Because Larch preserves the core XAFS analysis procedures of IFEFFIT with few algorithmic changes, data read this way is consistent with the IFEFFIT-lineage pipeline (normalization, background subtraction, Fourier transforms). Validate that the group is populated with finite energy and mu arrays over the expected energy range before passing it to autobk with documented settings such as rbkg=1.0 and kweight=2.

## Related tools

- **Larch (xraylarch)** (Provides the read_ascii function used to parse ASCII XAFS files into a data group for the XAFS processing workflow) — https://github.com/xraypy/xraylarch
- **Python** (Execution environment; Larch is written in Python and used as a Python library for scripted XAFS processing) — https://github.com/xraypy/xraylarch

## Examples

```
from larch.io import read_ascii; data = read_ascii('../xafsdata/fe.060')
```

## Evaluation signals

- The returned Larch group contains non-empty, finite energy and mu arrays parsed from the ASCII columns
- The subsequent autobk call on the group (rbkg=1.0, kweight=2) produces a chi(k) array populated with finite values
- Extracted fit parameters e0 and edge_step are present and physically plausible (cf. the article's Cu foil example: e0=8980.5 eV, edge_step=2.729)
- Output file such as fe060_autobk.csv exists and contains both the chi(k) array and the fit parameters

## Limitations

- read_ascii assumes well-formed column-based ASCII data; the context does not document automatic handling of headers, multi-column layouts, or malformed files
- Larch's GUI support covers XAS/XRF/XRD analysis, but some related modes (e.g., RIXS) have no processing GUI, so scripted use like this is required in those cases

## Evidence

- [results] Read ASCII XAFS data with read_ascii: "data = read_ascii('../xafsdata/fe.060')"
- [abstract] Larch preserves IFEFFIT XAFS procedures: "the initial release of larch preserves the core XAFS analysis procedures such as normalization, background subtraction, Fourier transforms, fitting of XANES spectra"
- [readme] Larch is an open-source XAS analysis library: "Larch is an open-source library and set of applications for visualizing, processing, and analyzing X-ray absorption and fluorescence spectroscopy data from synchrotron beamlines."
- [results] autobk applied after reading with documented settings: "autobk(dat.energy, dat.mu, rbkg=1.0, group=dat, clamp_hi=0)"
