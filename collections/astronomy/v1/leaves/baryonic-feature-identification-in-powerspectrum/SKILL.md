---
name: baryonic-feature-identification-in-powerspectrum
description: Use when you have the learned symbolic expression artifact for plin_emulated()
  and need to demonstrate that individual correction terms carry physical meaning
  (turnover, spectral index, acoustic/damping, baryonic effects) rather than acting
  as a black-box fit.
when_to_use_negative:
- You only need P(k) values for cosmological inference, not interpretation of the
  expression's structure — call plin_emulated() directly instead.
- The emulator is being used with extrapolate=False or outside the trained k range
  9e-3 to 9 h/Mpc, where the Eisenstein & Hu fit, not the learned symbolic terms,
  drives the output.
- You are interpreting the nonlinear emulator (SYREN-HALOFIT) or extended-cosmology
  emulators (SYREN-NEW, baryonic emulators) rather than the redshift-zero linear LCDM
  expression.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: numpy
  role: Evaluates each additive/multiplicative subterm of the learned expression over
    the k range and cosmology parameters to quantify contributions
- name: Python
  role: Language in which the symbolic_pofk emulators are provided and in which the
    subterm evaluation and JSON mapping are implemented
  repo: github.com/DeaglanBartlett/symbolic_pofk
- name: plin_emulated()
  role: ΛCDM linear emulator whose cosmology parameters and k range define the evaluation
    grid for subterm analysis
  repo: github.com/DeaglanBartlett/symbolic_pofk
- name: Eisenstein & Hu fit
  role: Baryon-inclusive baseline used for extrapolation outside the trained k range;
    reference point for annotating baryonic signatures
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:3b2a92ae55de15f68fd6cb1943ae3482474aa5d4038f91e4dd7d2916cdeb09b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eisenstein & Hu fit
  - Python
  - numpy
  - plin_emulated()
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# baryonic-feature-identification-in-powerspectrum

## Summary

Annotates each additive/multiplicative subterm of the symbolic_pofk linear matter power spectrum emulator (P(k), RMS fractional error 0.2% over k = 9e-3 to 9 h/Mpc) with its physical role — turnover scale, spectral index, acoustic/damping features, or baryonic signature — by evaluating subterm contributions over the trained k range and cosmology parameters, and emitting a structured JSON term-to-physics mapping.

## When to use

Use when you have the learned symbolic expression artifact for plin_emulated() and need to demonstrate that individual correction terms carry physical meaning (turnover, spectral index, acoustic/damping, baryonic effects) rather than acting as a black-box fit.

## When NOT to use

- You only need P(k) values for cosmological inference, not interpretation of the expression's structure — call plin_emulated() directly instead.
- The emulator is being used with extrapolate=False or outside the trained k range 9e-3 to 9 h/Mpc, where the Eisenstein & Hu fit, not the learned symbolic terms, drives the output.
- You are interpreting the nonlinear emulator (SYREN-HALOFIT) or extended-cosmology emulators (SYREN-NEW, baryonic emulators) rather than the redshift-zero linear LCDM expression.

## Inputs

- artifact_pk_expression — learned symbolic expression artifact from the symbolic_pofk package
- Emulator functions plin_emulated(), logF_max_precision(), logF_fiducial() from symbolic_pofk
- Emulator trained k range: 9e-3 to 9 h/Mpc, and emulator cosmology parameter grid
- Eisenstein & Hu baryon-inclusive fit (extrapolation baseline outside the trained k range)

## Outputs

- expression_term_mapping.json — structured JSON record with one entry per subterm: mathematical form, involved cosmological parameters, evaluated contribution range, and assigned physical role

## How to apply

Load the learned symbolic expression artifact (artifact_pk_expression) and the plin_emulated() function from the installed symbolic_pofk package, which was built by symbolic regression of the ratio between a low-accuracy fitting function and the Boltzmann-equation solution — so the learned correction terms inherit the physics of the original approximation. Using numpy, evaluate every additive/multiplicative subterm over the trained k range (9e-3 to 9 h/Mpc) and across the emulator's cosmology parameters to quantify each term's contribution and parameter dependence. Annotate each subterm with its physical role, cross-referencing the Eisenstein & Hu baryon-inclusive fit that the package uses as its extrapolation baseline outside the trained k range (note that this baseline is disabled with extrapolate=False in plin_emulated(), logF_max_precision() and logF_fiducial()). Assemble a structured JSON record (expression_term_mapping.json) with one entry per term holding the mathematical form, involved cosmological parameters, evaluated contribution range, and assigned physical role. Validate that the JSON parses, that every subterm in artifact_pk_expression maps to exactly one entry, and that each entry has a non-empty physical-role field with parameters matching those used in plin_emulated().

## Related tools

- **numpy** (Evaluates each additive/multiplicative subterm of the learned expression over the k range and cosmology parameters to quantify contributions)
- **Python** (Language in which the symbolic_pofk emulators are provided and in which the subterm evaluation and JSON mapping are implemented) — github.com/DeaglanBartlett/symbolic_pofk
- **plin_emulated()** (ΛCDM linear emulator whose cosmology parameters and k range define the evaluation grid for subterm analysis) — github.com/DeaglanBartlett/symbolic_pofk
- **Eisenstein & Hu fit** (Baryon-inclusive baseline used for extrapolation outside the trained k range; reference point for annotating baryonic signatures)

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git && cd symbolic_pofk && pip install . && python examples/linear_example.py
```

## Evaluation signals

- expression_term_mapping.json parses as valid JSON
- Every subterm appearing in artifact_pk_expression is covered by exactly one mapping entry (no orphan or duplicate mappings)
- Each JSON entry carries a non-empty physical-role field (turnover scale, spectral index, acoustic/damping feature, or baryonic signature)
- Cosmological parameters listed in each entry match the parameter set accepted by plin_emulated()
- Evaluated subterm contribution ranges fall within the trained k range 9e-3 to 9 h/Mpc and the emulator's cosmology parameter bounds

## Limitations

- The learned expression is only accurate within the trained k range (RMS fractional error 0.2% between k = 9e-3 and 9 h/Mpc); outside it the code falls back to the Eisenstein & Hu fit, which can be disabled with extrapolate=False.
- Term interpretations are inherited from the ratio-to-fitting-function construction; the symbolic regression learns corrections to an existing approximation, so physical annotations reflect that decomposition, not an independent derivation.
- This skill concerns the ΛCDM linear emulator at redshift zero; baryonic impact emulators and massive-neutrino/dynamical-dark-energy extensions (SYREN-NEW, syren-baryon) are separate models with their own citations.

## Evidence

- [abstract] Symbolic regression learns ratio to Boltzmann solution: "We learn the ratio between an existing low-accuracy fitting function forP(k) and that obtained by solving the Boltzmann equations"
- [abstract] RMS fractional error over trained k range: [evidence span withheld — anchor status: not_found]
- [readme] Eisenstein & Hu extrapolation baseline: "we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting `extrapolate=False` in the functions `plin_emulated()`"
- [readme] Linear emulator example script: "We give an example for how to use the ΛCDM linear emulator in `examples/linear_example.py`."
