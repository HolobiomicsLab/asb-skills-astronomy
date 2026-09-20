---
name: grid-resolution-convergence-tabulation
description: Use when a numerical-vs-exact solution pair is available at multiple
  grid resolutions and you need a tabulated, visual convergence indicator (FWHM of
  the fluctuation tending toward zero).
when_to_use_negative:
- Only one grid resolution (or one numerical/exact pair) is available — no FWHM-vs-resolution
  trend can be established
- The data are L1-norm error tables rather than pointwise profiles; the fluctuation-width
  method requires spatially resolved u_num and u_exact series
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3350
tools:
- name: aztekas
  role: Hydrodynamics code (finite-volume HRSC solver using PVRS) that generates the
    numerical density profiles at the six grid resolutions
  repo: https://github.com/aztekas-code/aztekas-main
- name: SciPy
  role: scipy.signal.peak_widths measures the FWHM of each fluctuation profile at
    half of its maximum
- name: NumPy
  role: Computes the pointwise normalized fluctuation |u_num − u_exact|/u_exact between
    numerical and exact density solutions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:a409c8be4b3bfe56ec31da2838408c1566b5f20241cd693083016aad85bd581a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - NumPy
  - Python
  - SciPy
  - aztekas
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3350
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# grid-resolution-convergence-tabulation

## Summary

Quantifies numerical convergence of a shock-capture scheme (here PVRS for 1D special relativistic hydrodynamics) by measuring how the FWHM of the pointwise normalized fluctuation |u_num − u_exact|/u_exact shrinks as grid resolution increases from Δx1=1/200 to Δx6=1/6400.

## When to use

Use when a numerical-vs-exact solution pair is available at multiple grid resolutions and you need a tabulated, visual convergence indicator (FWHM of the fluctuation tending toward zero).

## When NOT to use

- Only one grid resolution (or one numerical/exact pair) is available — no FWHM-vs-resolution trend can be established
- The data are L1-norm error tables rather than pointwise profiles; the fluctuation-width method requires spatially resolved u_num and u_exact series

## Inputs

- S1_File.tar.gz archive containing numerical and exact density profiles for Test 3 (strong relativistic blast wave) at Δx1=1/200, Δx2=1/400, Δx3=1/800, Δx4=1/1600, Δx5=1/3200, Δx6=1/6400

## Outputs

- fwhm_fluctuation_test3.csv — table of six FWHM values ordered by resolution (Δx1..Δx6)
- Per-resolution fluctuation profiles |u_num − u_exact|/u_exact

## How to apply

Unpack the deposited S1_File.tar.gz and load the stored numerical and exact density profiles for the strong relativistic blast wave (Test 3) at resolutions Δx1=1/200 through Δx6=1/6400. With NumPy/pandas compute the pointwise normalized fluctuation |u_num − u_exact|/u_exact between the numerical and exact density solutions for each resolution. Measure each fluctuation profile's Full Width at Half Maximum using scipy.signal.peak_widths evaluated at half of the fluctuation's maximum. Assemble the six FWHM values into a resolution-ordered table written to fwhm_fluctuation_test3.csv; acceptance requires a monotonic (or near-monotonic) decrease toward zero from Δx1 to Δx6, consistent with the paper's reported behavior.

## Related tools

- **aztekas** (Hydrodynamics code (finite-volume HRSC solver using PVRS) that generates the numerical density profiles at the six grid resolutions) — https://github.com/aztekas-code/aztekas-main
- **SciPy** (scipy.signal.peak_widths measures the FWHM of each fluctuation profile at half of its maximum)
- **NumPy** (Computes the pointwise normalized fluctuation |u_num − u_exact|/u_exact between numerical and exact density solutions)

## Examples

```
from scipy.signal import peak_widths; w = peak_widths(fluct, [int(fluct.argmax())], rel_height=0.5)
```

## Evaluation signals

- fwhm_fluctuation_test3.csv contains exactly six FWHM values, one per resolution Δx1..Δx6, ordered from 1/200 to 1/6400
- FWHM decreases monotonically (or near-monotonically) toward zero as resolution increases from Δx1 to Δx6
- Each FWHM is computed at half of that fluctuation profile's maximum via scipy.signal.peak_widths
- Fluctuation values are non-negative normalized ratios |u_num − u_exact|/u_exact, so no negative entries should appear in the profiles

## Limitations

- The FWHM is a visual/graphical convergence indicator of fluctuation width, not a quantitative convergence order (the paper uses L1-norm for that purpose)
- Monotonic decrease is expected but only 'near-monotonic' decrease is accepted as validation; isolated non-monotonic steps may occur
- Fluctuations are computed against exact analytic shock-tube solutions, so the method requires problems with known exact solutions

## Evidence

- [results] The Full Width at Half Maximum (FWHM) of the pointwise density fluctuation in Test 3 decreases monotonically toward zero as the grid resolution increases, providing a visual convergence indicator for the PVRS scheme.: [evidence span withheld — anchor status: not_found]
- [supplementary] In the file “S1_File.tar.gz”, we attached the relevant data of the comparison between numerical simulations and exact solutions used to obtain the L1—norm.: "In the file “S1_File.tar.gz”, we attached the relevant data of the comparison between numerical simulations and exact solutions"
- [abstract] We present the convergence of the method for standard shock-tube problems of special relativistic hydrodynamics and a graphical visualisation of the errors using the fluctuations of the numerical: "a graphical visualisation of the errors using the fluctuations of the numerical"
- [readme] _aztekas_ uses a conservative finite-volume approach to obtain the dicrete form of a hyperbolic partial differential system of equations (PDE).: "uses a conservative finite-volume approach to obtain the dicrete form of a hyperbolic partial differential system of equations"
