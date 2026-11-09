---
name: voigt-rheology-parameter-inversion
description: Use when a multilayered planetary interior model (radii, densities, shear
  moduli, viscosities in ModelInput.
when_to_use_negative:
- The body is already homogeneous or you need the full layered boundary-value solution
  at each frequency rather than a reduced surrogate — the inversion discards layer-by-layer
  structure by design.
- The rheology or forcing is outside the package's scope, e.g. compressible stratified
  bodies or non-tidal loading — the package computes the degree-2 tidal Love number
  for incompressible stratified viscoelastic bodies.
- You only need a layer cross-section figure — use VisualizeLayers.wl instead of the
  full inversion.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: wolframscript
  role: CLI driver used to run RunHomogeneous.wl, executing the pole search and inversion
  repo: github.com/ygevorgyan/HomogeneousModel
- name: Wolfram Language
  role: Implementation language of the package and configuration files (ModelInput.wl,
    src modules)
  repo: github.com/ygevorgyan/HomogeneousModel
- name: HomogeneousRheology.wl
  role: Core src/ module performing the equivalent homogeneous body computation (inversion
    into generalized Voigt compliance)
  repo: github.com/ygevorgyan/HomogeneousModel
- name: NLayerJacobian.wl
  role: N-layer propagator module providing k2(s) whose poles and residues feed the
    inversion
  repo: github.com/ygevorgyan/HomogeneousModel
- name: Mathematica 12.0+
  role: Alternative runtime to the free Wolfram Engine for executing the package
  repo: github.com/ygevorgyan/HomogeneousModel
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202659861
    title: A user-friendly package and workflow for generating effective homogeneous
      rheologies for the study of the long-term orbital evolution of multilayered
      planetary bodies
  source_content_hash: sha256:3d0970f6813e46c3e7029d630bfd74d74bab6467792ffa0dce936c3368b0ac3c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HomogeneousRheology.wl
  - Mathematica
  - Mathematica 12.0+
  - NLayerJacobian.wl
  - Wolfram Language
  - wolframscript
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# voigt-rheology-parameter-inversion

## Summary

Inverts the degree-2 tidal Love number response (k2) of a multilayered incompressible viscoelastic planetary body into an equivalent homogeneous generalized Voigt rheology using the Wolfram Language HomogeneousModel package, yielding Voigt parameters usable by external tidal-evolution codes.

## When to use

Use when a multilayered planetary interior model (radii, densities, shear moduli, viscosities in ModelInput.wl) must be reduced to an effective homogeneous generalized Voigt rheology so tidal dissipation can be included in long-term orbital/spin-evolution calculations without re-solving the layered boundary-value problem at each step.

## When NOT to use

- The body is already homogeneous or you need the full layered boundary-value solution at each frequency rather than a reduced surrogate — the inversion discards layer-by-layer structure by design.
- The rheology or forcing is outside the package's scope, e.g. compressible stratified bodies or non-tidal loading — the package computes the degree-2 tidal Love number for incompressible stratified viscoelastic bodies.
- You only need a layer cross-section figure — use VisualizeLayers.wl instead of the full inversion.

## Inputs

- ModelInput.wl configuration: $ModelConfig (rk, rhok, muk, etak arrays ordered center-to-surface, tidal frequency, optional layerNames) and $AnalysisConfig (workingPrecision, gridLogMin/gridLogMax/gridPoints, frequency range, dominant-mode threshold)
- Pre-configured test-case configurations (e.g. test_cases/ModelInputMoon.wl 5-layer Moon, ModelInputEuropa.wl, ModelInputIo.wl, ModelInputEnceladus.wl)
- Multilayered incompressible viscoelastic stratified body definition (consecutive liquid layers permitted; they are auto-merged)

## Outputs

- homogeneous_params.wl and homogeneous_params.json: Voigt parameters (elastic, gravitational, viscous, Voigt-element contributions) in formats usable by external tidal-evolution codes
- Love numbers (kE, kf, k2) and rheological parameters (gamma, alpha, eta) printed as terminal tables
- Secular relaxation modes, Voigt elements, and dominant-mode analysis
- Four comparison plots: real_k2_full.pdf, real_k2_reduced.pdf, imag_k2_full.pdf, imag_k2_reduced.pdf (N-layer vs full/reduced Voigt, Re[k2] and -Im[k2])

## How to apply

Configure the body in ModelInput.wl via $ModelConfig (radii, densities, shear moduli, viscosities, tidal frequency; liquid layers flagged with mu < 1 Pa, ordered center to surface) and $AnalysisConfig (precision, pole-search grid gridLogMin/gridLogMax/gridPoints, frequency range, dominant-mode threshold). Run 'wolframscript -file RunHomogeneous.wl' from homogeneous_model/; this performs the forward propagator-matrix computation of k2(s), a pole search over the log-grid to identify secular relaxation poles and residues, and inversion via the HomogeneousRheology module into the equivalent homogeneous generalized Voigt body. Treat FindRoot::brdig warnings as harmless; if fewer relaxation modes than expected are found, widen gridLogMin/gridLogMax or increase gridPoints, and if the N-layer and full-Voigt k2 curves separate in the overlay plots, raise workingPrecision to 80 or 100. Optionally run the dominant-mode selection procedure to obtain a reduced Voigt model preserving the tidal response over the frequency range relevant for orbital evolution.

## Related tools

- **wolframscript** (CLI driver used to run RunHomogeneous.wl, executing the pole search and inversion) — github.com/ygevorgyan/HomogeneousModel
- **Wolfram Language** (Implementation language of the package and configuration files (ModelInput.wl, src modules)) — github.com/ygevorgyan/HomogeneousModel
- **HomogeneousRheology.wl** (Core src/ module performing the equivalent homogeneous body computation (inversion into generalized Voigt compliance)) — github.com/ygevorgyan/HomogeneousModel
- **NLayerJacobian.wl** (N-layer propagator module providing k2(s) whose poles and residues feed the inversion) — github.com/ygevorgyan/HomogeneousModel
- **Mathematica 12.0+** (Alternative runtime to the free Wolfram Engine for executing the package) — github.com/ygevorgyan/HomogeneousModel

## Examples

```
cp test_cases/ModelInputMoon.wl ModelInput.wl && wolframscript -file RunHomogeneous.wl
```

## Evaluation signals

- Overlay plots of Re[k2] and -Im[k2] (N-layer vs full Voigt) in real_k2_full.pdf / imag_k2_full.pdf must overlap over the analysis frequency band with no visible separation
- Reduced Voigt model preserves the N-layer tidal response over the frequency interval relevant for orbital evolution with fewer relaxation elements
- Run completes in under 1 minute and produces homogeneous_params.wl / homogeneous_params.json plus the four comparison plots in output/
- Number of secular relaxation modes matches expectation; if fewer are found, widen gridLogMin/gridLogMax or increase gridPoints in $AnalysisConfig and re-run
- For merged stratified-liquid inputs, the three-sub-layer and pre-merged reductions agree to all printed digits

## Limitations

- FindRoot::brdig warnings appear during pole finding and are harmless, but poles can be missed if the gridLogMin/gridLogMax span or gridPoints count is too small
- Numerical precision can cause visible separation between N-layer and full-Voigt k2 curves; workingPrecision may need to be raised to 80 or 100
- Restricted to incompressible multilayered bodies under tidal forcing (degree-2 Love numbers); liquid layers are flagged by mu < 1 Pa
- Consecutive liquid layers are auto-merged by default (volume-weighted mean density); this preprocessing can be disabled with "MergeAdjacentLiquids" -> False but inputs with spurious sub-layer interfaces otherwise affect the propagator stack

## Evidence

- [abstract] workflow: forward Love numbers, pole identification, Voigt inversion: "We present a user-friendly, open-source Wolfram Language package that constructs an effective homogeneous generalized Voigt rheology"
- [abstract] lunar case study preserves tidal response: "we apply the package to a five-layer lunar interior model and obtain its equivalent generalized Voigt representation, together with a reduced model that preserves the tidal response over the"
- [methods] driver invocation: "Equivalent homogeneous body computation:

    wolframscript -file RunHomogeneous.wl"
- [methods] brdig warnings harmless: "FindRoot::brdig warnings are harmless."
- [readme] output files: "homogeneous_params.wl     Voigt parameters (Wolfram format)
  homogeneous_params.json   Voigt parameters (JSON)"
- [readme] troubleshooting precision: "If plots show separation between N-layer and full Voigt curves,
  increase workingPrecision to 80 or 100."
