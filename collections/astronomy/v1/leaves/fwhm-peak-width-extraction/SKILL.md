---
name: fwhm-peak-width-extraction
description: Use when you have stored numerical vs. exact solution profiles (e.g.,
  density in relativistic blast-wave Test 3) at several grid resolutions and need
  to show that the fluctuation width shrinks toward zero as Δx decreases.
when_to_use_negative:
- When profiles are already aggregated error metrics (e.g., L1-norms from S1_File.tar.gz)
  rather than pointwise fluctuation profiles — there is no peak to measure.
- When the goal is a formal convergence-order estimate from L1 norms instead of a
  visual/width-based convergence indicator.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: aztekas
  role: HRSC finite-volume solver used to generate the numerical relativistic hydrodynamics
    solutions whose fluctuations are analysed
  repo: https://github.com/aztekas-code/aztekas-main
- name: SciPy
  role: scipy.signal.peak_widths measures the FWHM of each fluctuation profile at
    half of the fluctuation's maximum
- name: NumPy
  role: computes the pointwise normalized fluctuation |u_num − u_exact|/u_exact between
    numerical and exact density solutions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:6022280460db8ed29777805504d575e806226fea68314789e8b6293af7f538b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - NumPy
  - Python
  - SciPy
  - aztekas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fwhm-peak-width-extraction

## Summary

Quantifies the spatial spread of pointwise density fluctuations between numerical and exact shock-tube solutions by measuring the Full Width at Half Maximum (FWHM) with scipy.signal.peak_widths, providing a visual/numerical convergence indicator for the PVRS scheme as grid resolution increases.

## When to use

Use when you have stored numerical vs. exact solution profiles (e.g., density in relativistic blast-wave Test 3) at several grid resolutions and need to show that the fluctuation width shrinks toward zero as Δx decreases.

## When NOT to use

- When profiles are already aggregated error metrics (e.g., L1-norms from S1_File.tar.gz) rather than pointwise fluctuation profiles — there is no peak to measure.
- When the goal is a formal convergence-order estimate from L1 norms instead of a visual/width-based convergence indicator.

## Inputs

- S1_File.tar.gz archive containing numerical and exact density profiles for Test 3 (strong relativistic blast wave)
- Grid resolution labels Δx1=1/200, Δx2=1/400, Δx3=1/800, Δx4=1/1600, Δx5=1/3200, Δx6=1/6400
- NumPy arrays of numerical density u_num and exact density u_exact per resolution

## Outputs

- Six FWHM values of the pointwise density fluctuation, one per resolution
- Resolution-ordered table written to fwhm_fluctuation_test3.csv

## How to apply

Unpack the deposited S1_File.tar.gz and load the numerical and exact density profiles for Test 3 at resolutions Δx1=1/200 through Δx6=1/6400 (six levels, factor-of-two refinement). For each resolution compute the pointwise normalized fluctuation |u_num − u_exact|/u_exact with NumPy/pandas, then measure the FWHM of each fluctuation profile using scipy.signal.peak_widths evaluated at half of the fluctuation's maximum. Assemble the six FWHM values into a resolution-ordered table (fwhm_fluctuation_test3.csv). The rationale, grounded in the article, is that the PVRS scheme's convergence for standard special-relativistic shock-tube problems is visualised via fluctuations of numerical values relative to exact analytic solutions, so a monotonic FWHM decrease toward zero evidences convergence.

## Related tools

- **aztekas** (HRSC finite-volume solver used to generate the numerical relativistic hydrodynamics solutions whose fluctuations are analysed) — https://github.com/aztekas-code/aztekas-main
- **SciPy** (scipy.signal.peak_widths measures the FWHM of each fluctuation profile at half of the fluctuation's maximum)
- **NumPy** (computes the pointwise normalized fluctuation |u_num − u_exact|/u_exact between numerical and exact density solutions)

## Examples

```
from scipy.signal import peak_widths; widths, _, _, _ = peak_widths(fluct, [int(fluct.argmax())], rel_height=0.5); print(widths[0]*dx)
```

## Evaluation signals

- FWHM values decrease monotonically (or near-monotonically) toward zero as resolution increases from Δx1=1/200 to Δx6=1/6400.
- Six FWHM entries are present, exactly one per resolution level, in fwhm_fluctuation_test3.csv.
- Each FWHM is measured at half of the fluctuation's maximum (rel_height corresponding to 0.5), matching the profile's own peak rather than a global threshold.
- Fluctuation profiles used for width measurement are normalized as |u_num − u_exact|/u_exact, not raw absolute differences.
- Profiles loaded from S1_File.tar.gz match the Test 3 strong relativistic blast-wave setup used in the paper's convergence visualisation.

## Limitations

- The validation criterion is monotonic or near-monotonic decrease, so a single non-monotonic FWHM value at one resolution may still be consistent with the paper's reported behavior and should not automatically fail the check.
- The FWHM is a visual convergence indicator, not a formal convergence rate; the paper's primary quantitative error measure is the L1-norm computed from the same deposited data.

## Evidence

- [abstract] We present the convergence of the method for standard shock-tube problems of special relativistic hydrodynamics and a graphical visualisation of the errors using the fluctuations of the numerical: "We present the convergence of the method for standard shock-tube problems of special relativistic hydrodynamics and a graphical visualisation of the errors using the fluctuations of the numerical"
- [supplementary] In the file "S1_File.tar.gz", we attached the relevant data of the comparison between numerical simulations and exact solutions used to obtain the L1—norm.: [evidence span withheld — anchor status: not_found]
- [readme] _aztekas_ is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes.: "_aztekas_ is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes."
