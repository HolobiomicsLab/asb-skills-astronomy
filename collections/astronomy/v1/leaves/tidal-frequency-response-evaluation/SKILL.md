---
name: tidal-frequency-response-evaluation
description: Use when an N-layer planetary interior model (radii, densities, shear
  moduli, viscosities, tidal frequency) must yield Re[k2] and -Im[k2] across a tidal-frequency
  range, or an equivalent Voigt rheology for long-term orbital-evolution codes.
when_to_use_negative:
- The goal is a fully compressible rheology or a nonlinear rheology — the package
  targets multilayered incompressible bodies under tidal forcing.
- Only a single-liquid-run input with no need for tidal-response computation over
  a frequency band is required and layer cross-section visualization alone suffices
  (use VisualizeLayers.wl instead).
- An input with several consecutive liquid sub-layers is being prepared and spurious
  internal interfaces must be removed — use the MergeAdjacentLiquids preprocessing
  decision rather than the response evaluation itself.
tools:
- name: Wolfram Language
  role: Implementation language for the package; requires Wolfram Engine (free) or
    Mathematica 12.0+ to run
  repo: github.com/ygevorgyan/HomogeneousModel
- name: wolframscript
  role: CLI driver for RunHomogeneous.wl, VisualizeLayers.wl and the k2(s) frequency
    sweep
  repo: github.com/ygevorgyan/HomogeneousModel
- name: YMatrix
  role: Constructs the 6x6 fundamental matrix and propagator for the incompressible
    stratified viscoelastic body
  repo: github.com/ygevorgyan/HomogeneousModel
- name: NLayerJacobian
  role: Evaluates the N-layer propagator k2(s) across the tidal-frequency range defined
    in $AnalysisConfig
  repo: github.com/ygevorgyan/HomogeneousModel
- name: HomogeneousRheology
  role: Computes the equivalent homogeneous generalized Voigt body, including optional
    dominant-mode reduction
  repo: github.com/ygevorgyan/HomogeneousModel
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202659861
    title: A user-friendly package and workflow for generating effective homogeneous
      rheologies for the study of the long-term orbital evolution of multilayered
      planetary bodies
  source_content_hash: sha256:4675e0b4f331066a2c8c85115704816401b3a5b5e9a9cff7d151c585c76dd10d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HomogeneousRheology
  - Mathematica
  - NLayerJacobian
  - Wolfram Language
  - YMatrix
  - wolframscript
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# tidal-frequency-response-evaluation

## Summary

Computes the degree-2 tidal Love number k2(s) frequency response of a multilayered incompressible viscoelastic body via the propagator-matrix formulation, and optionally an equivalent homogeneous generalized Voigt representation and a dominant-mode reduced model that preserve the tidal response over the orbital-evolution frequency band.

## When to use

Use when an N-layer planetary interior model (radii, densities, shear moduli, viscosities, tidal frequency) must yield Re[k2] and -Im[k2] across a tidal-frequency range, or an equivalent Voigt rheology for long-term orbital-evolution codes.

## When NOT to use

- The goal is a fully compressible rheology or a nonlinear rheology — the package targets multilayered incompressible bodies under tidal forcing.
- Only a single-liquid-run input with no need for tidal-response computation over a frequency band is required and layer cross-section visualization alone suffices (use VisualizeLayers.wl instead).
- An input with several consecutive liquid sub-layers is being prepared and spurious internal interfaces must be removed — use the MergeAdjacentLiquids preprocessing decision rather than the response evaluation itself.

## Inputs

- ModelInput.wl with $ModelConfig (layer arrays rk, rhok, muk, etak of equal length, ordered center-to-surface; liquid layers with mu < 1 Pa; tidal frequency)
- $AnalysisConfig (workingPrecision, pole-search gridLogMin/gridLogMax/gridPoints, frequency range, dominant-mode threshold)
- Pre-configured test cases such as test_cases/ModelInputMoon.wl (5-layer Moon), ModelInputEuropa.wl, ModelInputIo.wl, ModelInputEnceladus.wl

## Outputs

- Love numbers (kE, kf, k2), rheological parameters (gamma, alpha, eta), secular relaxation modes, Voigt elements, dominant-mode analysis
- output/homogeneous_params.wl and output/homogeneous_params. (Voigt parameters)
- output/real_k2_full.pdf, output/imag_k2_full.pdf (N-layer vs full Voigt comparisons)
- output/real_k2_reduced.pdf, output/imag_k2_reduced.pdf (N-layer vs reduced Voigt comparisons)
- Tabular k2 frequency response (e.g. nlayer_k2_frequency_response.csv with Re[k2] and -Im[k2])

## How to apply

Define the body in ModelInput.wl via $ModelConfig (radii rk, densities rhok, shear moduli muk, viscosities etak, tidal frequency; layers ordered center to surface, liquid layers flagged with mu < 1 Pa) and $AnalysisConfig (workingPrecision, pole-search grid gridLogMin/gridLogMax and gridPoints, frequency range, dominant-mode threshold). Load YMatrix.wl to build the 6x6 fundamental matrix and propagator and NLayerJacobian.wl to evaluate the N-layer k2(s) on the tidal-frequency grid, recording Re[k2] and -Im[k2]. Run 'wolframscript -file RunHomogeneous.wl' from homogeneous_model/ to identify secular relaxation poles and residues, invert into the equivalent homogeneous generalized Voigt body, and optionally apply dominant-mode selection to obtain a reduced Voigt model. Judge correctness by the N-layer and full-Voigt Re[k2] curves overlapping in output/real_k2_full.pdf and the reduced-Voigt curve overlaying in real_k2_reduced.pdf and imag_k2_reduced.pdf with no visible separation; if they separate, raise workingPrecision to 80 or 100, and if poles are missed, widen gridLogMin/gridLogMax or increase gridPoints.

## Related tools

- **Wolfram Language** (Implementation language for the package; requires Wolfram Engine (free) or Mathematica 12.0+ to run) — github.com/ygevorgyan/HomogeneousModel
- **wolframscript** (CLI driver for RunHomogeneous.wl, VisualizeLayers.wl and the k2(s) frequency sweep) — github.com/ygevorgyan/HomogeneousModel
- **YMatrix** (Constructs the 6x6 fundamental matrix and propagator for the incompressible stratified viscoelastic body) — github.com/ygevorgyan/HomogeneousModel
- **NLayerJacobian** (Evaluates the N-layer propagator k2(s) across the tidal-frequency range defined in $AnalysisConfig) — github.com/ygevorgyan/HomogeneousModel
- **HomogeneousRheology** (Computes the equivalent homogeneous generalized Voigt body, including optional dominant-mode reduction) — github.com/ygevorgyan/HomogeneousModel

## Examples

```
cp test_cases/ModelInputMoon.wl ModelInput.wl && wolframscript -file RunHomogeneous.wl
```

## Evaluation signals

- Re[k2] curves of the N-layer propagator solution and the full generalized Voigt model overlap in output/real_k2_full.pdf with no visible separation over the analysis frequency range
- Reduced Voigt curve overlays the N-layer curve in both output/real_k2_reduced.pdf and output/imag_k2_reduced.pdf over the prescribed frequency range
- No secular relaxation poles were missed on the pole-search grid (mode count matches expectations; widen gridLogMin/gridLogMax or increase gridPoints if fewer modes appear)
- The stratified-liquid reduction and the already-merged equivalent body agree to all printed digits
- FindRoot::brdig warnings appear during the pole search but are treated as harmless and do not invalidate the run

## Limitations

- Restricted to incompressible multilayered viscoelastic bodies under tidal forcing; compressibility is not modeled
- The pole-search grid (gridLogMin/gridLogMax, gridPoints) may miss relaxation poles if too narrow or coarse, requiring manual widening
- Numerical separation between N-layer and Voigt curves may require workingPrecision raised to 80 or 100
- The merge of consecutive liquid layers is on by default and can alter the layer stack unless disabled with "MergeAdjacentLiquids" -> False

## Evidence

- [abstract] Forward computation of degree-2 Love numbers and Voigt inversion: "a forward computation of the degree-2 tidal Love number based on the propagator-matrix formulation"
- [abstract] Lunar case study finding: "we apply the package to a five-layer lunar interior model and obtain its equivalent generalized Voigt representation"
- [intro] Configuration structure: "$ModelConfig      radii, densities, shear moduli, viscosities,
                    tidal frequency, optional layer names"
- [methods] Driver command: "Equivalent homogeneous body computation:

    wolframscript -file RunHomogeneous.wl"
- [readme] Precision troubleshooting: "If plots show separation between N-layer and full Voigt curves,
  increase workingPrecision to 80 or 100."
- [readme] Harmless warnings: "- FindRoot::brdig warnings are harmless."
- [readme] Output files: "real_k2_full.pdf          Re[k2]: N-layer vs full Voigt
  real_k2_reduced.pdf       Re[k2]: N-layer vs reduced Voigt"
