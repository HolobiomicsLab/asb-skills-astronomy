---
name: xafs-spline-spline-parameter-tuning
description: Use when a raw XAS/XAFS spectrum (e.g. energy vs. mu arrays from an ASCII
  data file such as fe.060) must be converted to chi(k) by background subtraction,
  or when autobk spline parameters like rbkg=1.0 and kweight=2 need adjustment.
when_to_use_negative:
- The spectrum has already been background-subtracted and you hold chi(k) — the next
  skill is Fourier transformation with xftf, not autobk
- The goal is XANES normalization or pre-edge fitting (pre_edge parameters like pre1/pre2/nnorm),
  not EXAFS spline background removal
- You are fitting EXAFS data to Feff paths via feffit with a feffit_transform (kmin/kmax/rmin/rmax);
  autobk is a prerequisite, not the fitting step
edam_operation: http://edamontology.org/operation_3799
tools:
- name: Larch (xraylarch)
  role: Provides the autobk function (from larch.xafs) that performs spline background
    subtraction producing chi(k), preserving IFEFFIT procedures
  repo: https://github.com/xraypy/xraylarch
- name: larch
  role: Package of XAFS analysis tools in which the autobk background-subtraction
    workflow is implemented
  repo: https://github.com/xraypy/xraylarch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:4375613185b77b72f2b2167f72a7c238e22f4386baf6ec87fcd1e3b3425d710c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xraypy/xraylarch
  tools:
  - Larch (xraylarch)
  - larch
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# xafs-spline-spline-parameter-tuning

## Summary

Tunes the spline-based background (autobk) parameters that convert a raw XAFS spectrum into chi(k), preserving the IFEFFIT background-subtraction procedure inside Larch. This step controls rbkg, kweight, and clamp settings that determine the smooth pre/post-edge spline removed from mu(E).

## When to use

Use when a raw XAS/XAFS spectrum (e.g. energy vs. mu arrays from an ASCII data file such as fe.060) must be converted to chi(k) by background subtraction, or when autobk spline parameters like rbkg=1.0 and kweight=2 need adjustment.

## When NOT to use

- The spectrum has already been background-subtracted and you hold chi(k) — the next skill is Fourier transformation with xftf, not autobk
- The goal is XANES normalization or pre-edge fitting (pre_edge parameters like pre1/pre2/nnorm), not EXAFS spline background removal
- You are fitting EXAFS data to Feff paths via feffit with a feffit_transform (kmin/kmax/rmin/rmax); autobk is a prerequisite, not the fitting step

## Inputs

- Raw XAFS spectrum as energy vs. mu arrays from an ASCII data file (e.g. fe.060 from the xafsdata folder)
- Documented autobk settings: rbkg (e.g. 1.0), kweight (e.g. 2), optional clamp_hi (e.g. 0, 50, 200)

## Outputs

- chi(k) array stored in the Larch data group
- Autobk fit parameters in the data group: e0, edge_step, and the spline background
- Exported table of chi(k) and fit parameters (e.g. fe060_autobk.csv)

## How to apply

Load the raw spectrum with Larch's read_ascii (e.g. data = read_ascii('../xafsdata/fe.060')), then run larch.xafs.autobk on the data group's energy and mu arrays with documented settings such as rbkg=1.0 and kweight=2, which writes the chi(k) result and fit parameters (e0, edge_step, spline background) into the data group. Larch's initial release deliberately preserves the core IFEFFIT XAFS analysis procedures, including background subtraction, so parameters follow IFEFFIT conventions rather than new algorithms. To probe spline rigidity sensitivity, rerun autobk with varying clamp_hi values (e.g. 0, 50, 200) and compare the resulting chi(k). Decision point: choose rbkg small enough to follow genuine background oscillations but large enough not to absorb low-k EXAFS signal; clamp_hi suppresses high-end spline wobble. Export chi(k) and fit parameters (e.g. to fe060_autobk.csv) and validate that chi(k) is finite and parameters are populated before proceeding to xftf Fourier transforms.

## Related tools

- **Larch (xraylarch)** (Provides the autobk function (from larch.xafs) that performs spline background subtraction producing chi(k), preserving IFEFFIT procedures) — https://github.com/xraypy/xraylarch
- **larch** (Package of XAFS analysis tools in which the autobk background-subtraction workflow is implemented) — https://github.com/xraypy/xraylarch

## Examples

```
from larch.xafs import autobk; autobk(dat.energy, dat.mu, rbkg=1.0, group=dat, clamp_hi=0)
```

## Evaluation signals

- The output file (e.g. fe060_autobk.csv) exists and the chi(k) array it contains is populated with finite values
- Extracted fit parameters e0, edge_step, and the spline background are present and numerically populated in the data group
- The autobk call used the documented setting rbkg=1.0 (optionally kweight=2, clamp_hi=10), matching the preserved IFEFFIT procedure
- chi(k) oscillations are visible at low k without strong residual background; varying clamp_hi (0/50/200) changes only the high-k spline behavior, not the core chi(k) structure

## Limitations

- Spline parameter choice (rbkg, clamp_hi) is a sensitivity point: the article's autobk sensitivity test with clamp_hi values 0, 50, and 200 shows results depend on these settings
- Larch's initial release preserves IFEFFIT algorithms with few changes, so new background-subtraction algorithms are out of scope for this skill
- Validation is limited to confirming finite chi(k) values and presence of fit parameters; the workflow does not guarantee physically optimal parameter choices for arbitrary spectra

## Evidence

- [results] larch.xafs imports: [evidence span withheld — anchor status: not_found]
