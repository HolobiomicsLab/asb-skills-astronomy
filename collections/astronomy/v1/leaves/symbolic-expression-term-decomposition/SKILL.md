---
name: symbolic-expression-term-decomposition
description: Use when you have a fitted symbolic expression (e.g. artifact_pk_expression
  from symbolic_pofk) and need to determine whether individual terms map onto physical
  features like the turnover scale, spectral index, acoustic/damping features, or
  baryonic signatures.
when_to_use_negative:
- You only need to evaluate P(k) to desired accuracy — use plin_emulated() directly
  rather than decomposing terms
- The expression is a black-box numeric surrogate (e.g. a neural network emulator
  like BACCO) with no closed-form subterms to isolate
- You are working outside the trained k range, where the code extrapolates with the
  Eisenstein & Hu fit instead of the learned symbolic correction
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: symbolic_pofk
  role: Provides the learned linear P(k) expression artifact and plin_emulated() emulator
    whose subterms are decomposed and annotated
  repo: https://github.com/DeaglanBartlett/symbolic_pofk
- name: numpy
  role: Evaluates each subterm of the symbolic expression over the k range and cosmology
    parameters to quantify contribution ranges
- name: Eisenstein & Hu fit
  role: Baryon-inclusive baseline fit used by plin_emulated() for extrapolation outside
    the trained k range; the physical-role annotation cross-references it
- name: Python
  role: Language in which the emulator functions and the decomposition/annotation
    workflow are implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:ac5f64df37dc605b1b46f201c6e817948731feb03e1334cd9ca47ca6e9d58f1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DeaglanBartlett/symbolic_pofk
  tools:
  - Eisenstein & Hu fit
  - Python
  - numpy
  - symbolic_pofk
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# symbolic-expression-term-decomposition

## Summary

Decomposes a learned symbolic regression expression (e.g. the symbolic_pofk emulator for the linear matter power spectrum) into its additive/multiplicative subterms, evaluating each term's contribution and cosmological-parameter dependence to assign physical roles and demonstrate the expression is interpretable rather than a black-box fit.

## When to use

Use when you have a fitted symbolic expression (e.g. artifact_pk_expression from symbolic_pofk) and need to determine whether individual terms map onto physical features like the turnover scale, spectral index, acoustic/damping features, or baryonic signatures.

## When NOT to use

- You only need to evaluate P(k) to desired accuracy — use plin_emulated() directly rather than decomposing terms
- The expression is a black-box numeric surrogate (e.g. a neural network emulator like BACCO) with no closed-form subterms to isolate
- You are working outside the trained k range, where the code extrapolates with the Eisenstein & Hu fit instead of the learned symbolic correction

## Inputs

- Learned symbolic expression artifact (artifact_pk_expression) from the symbolic_pofk linear P(k) emulator
- Emulator function plin_emulated() from the installed symbolic_pofk package
- Trained k-range bounds (9e-3 to 9 h/Mpc) and the emulator's cosmology parameter grid
- Eisenstein & Hu baryon-inclusive fit used as the extrapolation baseline

## Outputs

- expression_term_mapping.json: one entry per subterm with mathematical form, cosmological parameters, evaluated contribution range, and assigned physical role
- Per-term contribution and parameter-dependence profiles (numpy evaluations)

## How to apply

Load the learned symbolic expression artifact (artifact_pk_expression) and the emulator function plin_emulated() from the installed symbolic_pofk package. Evaluate each additive/multiplicative subterm with numpy over the trained k range (9e-3 to 9 h/Mpc) and across the emulator's cosmology parameters to quantify each term's contribution range and parameter dependence. Annotate each subterm with its physical role — turnover scale, spectral index, acoustic/damping features, or baryonic signature — cross-referencing the Eisenstein & Hu baryon-inclusive fit that the code uses as the extrapolation baseline outside the trained k range. Assemble a structured JSON record (expression_term_mapping.json) with one entry per term containing the mathematical form, involved cosmological parameters, evaluated contribution range, and assigned physical role. The interpretability rationale is grounded in the construction: symbolic regression learns the ratio between a low-accuracy fitting function and the Boltzmann-equation solution, so correction terms inherit and extend the original approximation's physics.

## Related tools

- **symbolic_pofk** (Provides the learned linear P(k) expression artifact and plin_emulated() emulator whose subterms are decomposed and annotated) — https://github.com/DeaglanBartlett/symbolic_pofk
- **numpy** (Evaluates each subterm of the symbolic expression over the k range and cosmology parameters to quantify contribution ranges)
- **Eisenstein & Hu fit** (Baryon-inclusive baseline fit used by plin_emulated() for extrapolation outside the trained k range; the physical-role annotation cross-references it)
- **Python** (Language in which the emulator functions and the decomposition/annotation workflow are implemented)

## Examples

```
from symbolic_pofk import plin_emulated; import numpy as np; k = np.logspace(-3+np.log10(9), np.log10(9), 200); P = plin_emulated(k, As=2.1e-9, ns=0.965, H0=67.0, Om=0.31, Ob=0.049)
```

## Evaluation signals

- expression_term_mapping.json parses as valid JSON
- Every subterm appearing in artifact_pk_expression is covered by exactly one mapping entry (no orphans, no duplicates)
- Each entry carries a non-empty physical-role field
- Cosmological parameters listed in each entry match those used in plin_emulated()
- Evaluated contribution ranges fall within the trained k range 9e-3 to 9 h/Mpc and reproduce plin_emulated() output when recombined

## Limitations

- Physical-role assignment relies on the construction (learning the ratio to a low-accuracy fitting function); correction terms only 'inherit and extend' the original approximation's physics, so some terms may resist clean physical labels
- Term interpretation is valid only within the trained k range; outside it the emulator switches to the Eisenstein & Hu extrapolation and the learned correction no longer applies
- Interpretability claims are for the ΛCDM linear emulator of Bartlett et al. 2023; extended-cosmology (neutrino masses, w0, wa) or baryonic emulators are separate models with different expressions

## Evidence

- [other] Interpretability stems from construction: [evidence span withheld — anchor status: not_found]
- [abstract] Learned ratio to Boltzmann solution: "We learn the ratio between an existing low-accuracy fitting function forP(k) and that obtained by solving the Boltzmann equations"
- [readme] Eisenstein & Hu extrapolation baseline: "we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting
`extrapolate=False` in the functions `plin_emulated()`, `logF_max_precision()` and
`logF_fiducial()`."
- [abstract] Trained k range and accuracy: [evidence span withheld — anchor status: not_found]
- [intro] Emulators provided as Python functions: "Here we give the emulators as simple python functions. 
Some emulators are given as torch and/or fortran90 functions, but all can be 
easily copied, pasted and modified to the language of your choice."
