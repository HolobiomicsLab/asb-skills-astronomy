---
name: exafs-fit-statistics-extraction
description: Use when a feffit refinement of Feff paths to EXAFS chi(k) data has completed
  and you need to pull refined parameter values, chi-square, and R-factor out of the
  results object for model comparison or reporting (e.g. to a CSV).
when_to_use_negative:
- You are still processing raw mu(E) data (normalization, autobk background subtraction)
  — extraction applies only after feffit has actually refined path parameters.
- The candidate models were fitted with different feffit_transform settings (different
  kmin/kmax, kweight, or r-range) — chi-square and R-factor are then not comparable.
- Fitting XANES spectra or linear-combination/PCA analysis rather than EXAFS path
  fitting — no Feff paths or feffit results exist to extract statistics from.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3418
tools:
- name: Larch (xraylarch)
  role: provides feffit and feffit_transform for refining path parameters and reporting
    chi-square and R-factor
  repo: github.com/xraypy/xraylarch
- name: feff
  role: computes the photoelectron scattering paths that feffit refines against the
    data
- name: Python
  role: language in which the feffit results are extracted and written to CSV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1088/1742-6596/430/1/012007
    title: 'Larch: An Analysis Package for XAFS and Related Spectroscopies'
  source_content_hash: sha256:823b72162e505501a87857590eeed68a1a3661f9071a979ce5014c58eb022726
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Larch (xraylarch)
  - Python
  - feff
  - larch
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# exafs-fit-statistics-extraction

## Summary

Extract refined fit parameters, chi-square, and R-factor from Larch feffit results after refining Feff path parameters against EXAFS chi(k) data (e.g. the Zn K-edge of ZnSe), and tabulate them per structural model so competing fits can be compared.

## When to use

Use when a feffit refinement of Feff paths to EXAFS chi(k) data has completed and you need to pull refined parameter values, chi-square, and R-factor out of the results object for model comparison or reporting (e.g. to a CSV).

## When NOT to use

- You are still processing raw mu(E) data (normalization, autobk background subtraction) — extraction applies only after feffit has actually refined path parameters.
- The candidate models were fitted with different feffit_transform settings (different kmin/kmax, kweight, or r-range) — chi-square and R-factor are then not comparable.
- Fitting XANES spectra or linear-combination/PCA analysis rather than EXAFS path fitting — no Feff paths or feffit results exist to extract statistics from.

## Inputs

- Experimental EXAFS chi(k) data (e.g. Zn K-edge of ZnSe)
- Feff-computed scattering paths assembled via feffpath (from feffrunner runs on candidate structures, e.g. AMCSD 11535 cubic and AMCSD 11557 hexagonal ZnSe)
- feffit_transform configuration (kmin=3, kmax=12, kweight=2, dk=4, window='kaiser', rmin=1.0, rmax=3.0 in the ZnSe example)

## Outputs

- feffit results object containing refined path parameters, chi-square, and R-factor
- CSV summary of fit statistics per structural model (e.g. feffit_hexagonal_znse_results.csv)

## How to apply

After assembling Feff paths with feffpath and building the theoretical chi(k) with ff2chi, define the fit transform (in the ZnSe example: feffit_transform with kmin=3, kmax=12, kweight=2, dk=4, window='kaiser', rmin=1.0, rmax=3.0) and run feffit against the experimental chi(k) — here the Zn K-edge of ZnSe. From the returned feffit results object, read out the refined path parameters plus the chi-square and R-factor goodness-of-fit statistics. Repeat the identical transform and fitting recipe for each candidate structural model (cubic AMCSD 11535, F -4 3 m; hexagonal AMCSD 11557, P 63 m c) so the statistics are directly comparable. Write the extracted values to a table such as feffit_hexagonal_znse_results.csv, and judge model quality by which fit gives the lower R-factor/chi-square for the same k-range and R-range window.

## Related tools

- **Larch (xraylarch)** (provides feffit and feffit_transform for refining path parameters and reporting chi-square and R-factor) — github.com/xraypy/xraylarch
- **feff** (computes the photoelectron scattering paths that feffit refines against the data)
- **Python** (language in which the feffit results are extracted and written to CSV)

## Examples

```
from larch.xafs import feffit, feffit_transform; trans = feffit_transform(kmin=3, kmax=12, kweight=2, dk=4, window='kaiser', rmin=1.0, rmax=3.0); out = feffit(trans, paths, params, data=zn_data); print(out.chi_square, out.rfactor)
```

## Evaluation signals

- The results object exposes non-negative R-factor and finite chi-square after feffit converges.
- Refined path parameter values are physically sensible (e.g. bond lengths near the crystallographic Zn-Se distances of the cubic/hexagonal CIF models).
- The same feffit_transform (kmin=3, kmax=12, kweight=2, dk=4, kaiser, rmin=1.0, rmax=3.0) is used for every model being compared.
- The CSV (e.g. feffit_hexagonal_znse_results.csv) contains one row per model with parameter values, chi-square, and R-factor columns populated.
- The model with the better match to the Zn K-edge data shows the lower R-factor under the identical transform.

## Limitations

- R-factor and chi-square are only meaningful within a fixed fit transform; changing k-weight or k-range invalidates cross-model comparison.
- Extraction depends on Feff having produced the path set — Larch's CIF-to-Feff conversion for external structures has had fixes applied per its changelog, so path generation for unusual structures can fail.
- Feff8l CLI computes EXAFS only (no XANES), so statistics extraction is limited to EXAFS path fits.

## Evidence

- [results] feffit transform ranges: kmin=3, kmax=12, kweight=2, dk=4, kaiser window, rmin=1.0, rmax=3.0: "trans = feffit_transform(kmin=3, kmax=12, kweight=2, dk=4, window='kaiser', rmin=1.0, rmax=3.0)"
- [other] Workflow step 6: extract fit parameters and statistics: [evidence span withheld — anchor status: not_found]
- [other] Two structural models compared by fitting the same data: [evidence span withheld — anchor status: not_found]
- [results] AMCSD structures returned for the two ZnSe models: [evidence span withheld — anchor status: not_found]
- [readme] feffit role in Larch from README: [evidence span withheld — anchor status: external_doc]
- [abstract] Core XAFS fitting procedures preserved in Larch: "fitting of experimental spectra to a sum of feff Paths"
