---
name: cosmological-parameter-interpretation
description: Use when a learned symbolic expression for the linear matter power spectrum
  (or similar emulator formula) needs each subterm annotated with its physical role
  and cosmological-parameter dependence, e.g. to show interpretability beyond black-box
  fitting.
when_to_use_negative:
- The goal is numerical accuracy benchmarking of the emulator (e.g. RMS fractional
  error or speed vs CAMB/BACCO) rather than term-level physical interpretation.
- The expression under study is not built as a learned ratio over a physics-motivated
  fitting function — correction terms will not inherit interpretable physics.
- Working outside the trained k range (k < 9e-3 or k > 9 h/Mpc) with extrapolate=True,
  where the Eisenstein & Hu baseline rather than the learned terms dominates.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3298
tools:
- name: numpy
  role: Numerically evaluates each additive/multiplicative subterm of the learned
    expression over the trained k range and cosmology parameters to quantify contribution
    ranges
- name: symbolic_pofk
  role: Provides artifact_pk_expression and plin_emulated(); installed via git clone
    + pip install .
  repo: https://github.com/DeaglanBartlett/symbolic_pofk
- name: Eisenstein & Hu fit
  role: Baryon-inclusive baseline fit used for extrapolation outside the trained k
    range; serves as the physical cross-reference for annotating term roles
- name: Python
  role: Language in which the emulators are delivered as simple functions and in which
    the term-by-term evaluation and JSON assembly are scripted
  repo: https://github.com/DeaglanBartlett/symbolic_pofk
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:b755aff652dbfebac94c231c3d6a962fb5db0cd8045fc017711fb8846c71ca0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DeaglanBartlett/symbolic_pofk
  tools:
  - Eisenstein & Hu fit
  - Python
  - numpy
  - symbolic_pofk
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3298
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cosmological-parameter-interpretation

## Summary

A method for assigning physical roles (turnover scale, spectral index, acoustic/damping features, baryonic signatures) to individual additive/multiplicative subterms of the symbolic emulator's learned expression for the linear matter power spectrum, demonstrating the expression is not a black-box fit.

## When to use

Use when a learned symbolic expression for the linear matter power spectrum (or similar emulator formula) needs each subterm annotated with its physical role and cosmological-parameter dependence, e.g. to show interpretability beyond black-box fitting.

## When NOT to use

- The goal is numerical accuracy benchmarking of the emulator (e.g. RMS fractional error or speed vs CAMB/BACCO) rather than term-level physical interpretation.
- The expression under study is not built as a learned ratio over a physics-motivated fitting function — correction terms will not inherit interpretable physics.
- Working outside the trained k range (k < 9e-3 or k > 9 h/Mpc) with extrapolate=True, where the Eisenstein & Hu baseline rather than the learned terms dominates.

## Inputs

- artifact_pk_expression: the learned symbolic expression for the linear matter power spectrum from the symbolic_pofk package
- plin_emulated(): the LCDM linear emulator function (with extrapolate flag controlling Eisenstein & Hu extrapolation)
- Trained k range: k = 9e-3 to 9 h/Mpc
- Emulator cosmology parameter grid

## Outputs

- expression_term_mapping.json: one entry per subterm with mathematical form, involved cosmological parameters, evaluated contribution range, and assigned physical role

## How to apply

Load the learned symbolic expression artifact (artifact_pk_expression) and the corresponding emulator function plin_emulated() from the installed symbolic_pofk package. Using numpy, evaluate each additive/multiplicative subterm of the expression over the trained k range (9e-3 to 9 h/Mpc) and across the emulator's cosmology parameters to quantify each term's contribution and parameter dependence. Annotate every subterm with its physical role — turnover scale, spectral index, acoustic/damping features, or baryonic signature — cross-referencing the Eisenstein & Hu baryon-inclusive fit that the code uses as its extrapolation baseline outside the trained k range. The interpretability is grounded in construction: the symbolic regression learns the ratio between a low-accuracy fitting function for P(k) and the Boltzmann-equation solution, so correction terms inherit the physics motivating the original approximation. Assemble a structured JSON record (expression_term_mapping.json) with one entry per term containing the mathematical form, involved cosmological parameters, evaluated contribution range, and assigned physical role.

## Related tools

- **numpy** (Numerically evaluates each additive/multiplicative subterm of the learned expression over the trained k range and cosmology parameters to quantify contribution ranges)
- **symbolic_pofk** (Provides artifact_pk_expression and plin_emulated(); installed via git clone + pip install .) — https://github.com/DeaglanBartlett/symbolic_pofk
- **Eisenstein & Hu fit** (Baryon-inclusive baseline fit used for extrapolation outside the trained k range; serves as the physical cross-reference for annotating term roles)
- **Python** (Language in which the emulators are delivered as simple functions and in which the term-by-term evaluation and JSON assembly are scripted) — https://github.com/DeaglanBartlett/symbolic_pofk

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git; cd symbolic_pofk; pip install .; then in Python: from symbolic_pofk import plin_emulated; P = plin_emulated(k, params, extrapolate=False)
```

## Evaluation signals

- expression_term_mapping.json parses as valid JSON
- Every subterm appearing in artifact_pk_expression is covered by exactly one mapping entry (no orphan or duplicated terms)
- Each JSON entry carries a non-empty physical-role field
- Cosmological parameters listed in each entry match those used by plin_emulated()
- Per-term contribution ranges are finite and evaluated strictly over k = 9e-3 to 9 h/Mpc

## Limitations

- Interpretability relies on the construction: symbolic regression learns the ratio between a low-accuracy fitting function and the Boltzmann-equation solution; expressions built differently may not admit physical interpretation.
- Outside the trained k range (9e-3 to 9 h/Mpc), the default expression extrapolates via the Eisenstein & Hu baryon-inclusive fit, so term-level physical roles from the learned correction do not apply there; set extrapolate=False to isolate the learned expression.
- Assigned physical roles (turnover, spectral index, acoustic/damping, baryonic signatures) are annotations informed by the baseline physics, not independently derived quantities.

## Evidence

- [abstract] Interpretability stems from construction: regression learns ratio between low-accuracy fitting function and Boltzmann solution: "We learn the ratio between an existing low-accuracy fitting function forP(k) and that obtained by solving the Boltzmann equations"
- [methods] Eisenstein & Hu baryon-inclusive fit as extrapolation baseline: "we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting extrapolate=False"
- [abstract] Trained k-range bounds for term evaluation: [evidence span withheld — anchor status: not_found]
- [intro] Emulators delivered as Python functions supporting term-level evaluation: "Here we give the emulators as simple python functions."
- [readme] README confirms extrapolation behavior of plin_emulated(): "outside the k range tested in Bartlett et al. 2023, 
we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting
`extrapolate=False` in the functions `plin_emulated()`,"
