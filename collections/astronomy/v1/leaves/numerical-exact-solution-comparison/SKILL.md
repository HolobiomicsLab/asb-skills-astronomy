---
name: numerical-exact-solution-comparison
description: Use when you have numerical solutions of a conservation law (e.g. SRHD
  shock tubes) at several grid resolutions plus exact solution data, and you need
  to measure convergence, error norms, or fluctuation widths.
when_to_use_negative:
- When no exact analytic solution is available — the fluctuation and L1 diagnostics
  both require u_exact as the reference
- 'When validating the non-conservative FDM-only primitive update (Eq 10) on solutions
  containing shocks: by design it is ''stable but not consistent and so no convergent'',
  so expecting monotonic error decrease is the wrong criterion — the expected result
  is non-convergence to a different weak solution'
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: aztekas
  role: HRSC finite-volume solver used to generate the numerical density profiles
    that are compared against exact solutions
  repo: https://github.com/aztekas-code/aztekas-main
- name: SciPy
  role: scipy.signal.peak_widths measures the FWHM of the pointwise fluctuation profile
    at half of the fluctuation's maximum
- name: NumPy
  role: computes the pointwise normalized fluctuation |u_num − u_exact|/u_exact and
    L1-norm errors between numerical and exact density arrays
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:75ddb1e524af2cb352eec8b1c40f3748e16d1f605cac9ce4711a7a9a9712904b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - NumPy
  - Python
  - SciPy
  - aztekas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# numerical-exact-solution-comparison

## Summary

Quantifies convergence of a numerical scheme for 1D relativistic hydrodynamics by comparing stored numerical profiles against exact Riemann/analytics solutions from S1_File.tar.gz, using L1-norm errors, per-resolution convergence orders, and Full Width at Half Maximum (FWHM) of pointwise normalized fluctuations.

## When to use

Use when you have numerical solutions of a conservation law (e.g. SRHD shock tubes) at several grid resolutions plus exact solution data, and you need to measure convergence, error norms, or fluctuation widths.

## When NOT to use

- When no exact analytic solution is available — the fluctuation and L1 diagnostics both require u_exact as the reference
- When validating the non-conservative FDM-only primitive update (Eq 10) on solutions containing shocks: by design it is 'stable but not consistent and so no convergent', so expecting monotonic error decrease is the wrong criterion — the expected result is non-convergence to a different weak solution

## Inputs

- Numerical density (or other primitive variable) profiles from aztekas/conservative scheme runs at resolutions Δx = 1/200, 1/400, 1/800, 1/1600, 1/3200, 1/6400
- Exact Riemann/analytic solution profiles shipped in S1_File.tar.gz
- Resolution values and common final simulation time

## Outputs

- CSV table of per-resolution error metrics (fwhm_fluctuation_test3.csv or fdm_l1_errors.csv)
- Plot of numerical vs exact density profiles (e.g. fdm_sod_profile.png)
- Convergence orders log(error_i/error_{i-1})/log(1/2) per resolution pair

## How to apply

Load the numerical and exact density profiles from the deposited S1_File.tar.gz for a given test (e.g. Test 3, the strong relativistic blast wave) at resolutions Δx1=1/200 through Δx6=1/6400. Compute the pointwise normalized fluctuation |u_num − u_exact|/u_exact, then either (a) compute the L1-norm error per resolution and the convergence order log(error_i/error_{i-1})/log(1/2), interpolating runs onto the exact solution data, or (b) measure the FWHM of each fluctuation profile with scipy.signal.peak_widths evaluated at half of the fluctuation's maximum. Assemble the per-resolution values into a table (e.g. fwhm_fluctuation_test3.csv or fdm_l1_errors.csv) and plot numerical vs exact profiles. Rationale: the paper uses these fluctuation and L1 diagnostics to demonstrate that the PVRS scheme converges on standard shock-tube problems, while the non-conservative FDM-only discretisation of Eq (10) does not, because it violates the entropy and Rankine-Hugoniot jump conditions at shocks.

## Related tools

- **aztekas** (HRSC finite-volume solver used to generate the numerical density profiles that are compared against exact solutions) — https://github.com/aztekas-code/aztekas-main
- **SciPy** (scipy.signal.peak_widths measures the FWHM of the pointwise fluctuation profile at half of the fluctuation's maximum)
- **NumPy** (computes the pointwise normalized fluctuation |u_num − u_exact|/u_exact and L1-norm errors between numerical and exact density arrays)

## Examples

```
from scipy.signal import peak_widths; widths, _, _, _ = peak_widths(fluctuation, [peak_index], rel_height=0.5)
```

## Evaluation signals

- The tabulated FWHM of the pointwise density fluctuation decreases monotonically (or near-monotonically) toward zero as resolution increases from Δx1 to Δx6
- For the conservative PVRS scheme, L1-norm errors decrease monotonically with resolution with positive convergence orders log(error_i/error_{i-1})/log(1/2)
- As a negative control, the FDM-only Eq (10) scheme shows L1 errors that fail to decrease monotonically, with convergence order near zero at the shock
- Output CSV contains exactly one error/FWHM value per resolution and is ordered by increasing resolution
- Plotted numerical profiles overlay the exact Riemann solution from S1_File.tar.gz, with residual fluctuations shrinking at higher resolution

## Limitations

- The exact solution data must be interpolated onto each numerical grid before computing the L1 norm, so interpolation error can contaminate the lowest-resolution runs
- The FDM-only discretisation of the primitive-variable equation (Eq 10) is non-conservative and does not satisfy the entropy and Rankine-Hugoniot jump conditions, so it converges to a different weak solution than a conservative method — this comparison skill must not be used to 'validate' that scheme on shock problems
- The validation criterion in the source card accepts near-monotonic FWHM decrease, not strict monotonicity, so isolated non-monotonic steps between adjacent resolutions are tolerable

## Evidence

- [supplementary] In the file “S1_File.tar.gz”, we attached the relevant data of the comparison between numerical simulations and exact solutions used to obtain the L1—norm.: "In the file “S1_File.tar.gz”, we attached the relevant data of the comparison between numerical simulations and exact solutions used to obtain the L1—norm."
- [abstract] We present the convergence of the method for standard shock-tube problems of special relativistic hydrodynamics and a graphical visualisation of the errors using the fluctuations: "We present the convergence of the method for standard shock-tube problems of special relativistic hydrodynamics and a graphical visualisation of the errors using the fluctuations"
- [readme] _aztekas_ uses a conservative finite-volume approach to obtain the dicrete form of a hyperbolic partial differential system of equations (PDE).: "_aztekas_ uses a conservative finite-volume approach to obtain the dicrete form of a hyperbolic partial differential system of equations (PDE)."
- [abstract] a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions: "a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions"
