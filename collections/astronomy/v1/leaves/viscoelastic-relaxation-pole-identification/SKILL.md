---
name: viscoelastic-relaxation-pole-identification
description: Use when a layered planetary interior model (radii, densities, shear
  moduli, viscosities in ModelInput.wl) must be reduced to an effective homogeneous
  Voigt rheology for long-term tidal/orbital evolution calculations.
when_to_use_negative:
- The goal is a full time-domain viscoelastic relaxation spectrum solve rather than
  an equivalent homogeneous rheology for orbital-evolution codes
- The body model is a single homogeneous layer with no stratification, so no layered
  pole/residue identification is needed
- The input contains already at most one liquid run and the user only needs layer
  visualization (VisualizeLayers.wl suffices)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Wolfram Language
  role: Language in which the package's propagator-matrix k2(s) computation, pole/residue
    search, and HomogeneousRheology inversion are implemented
  repo: https://github.com/ygevorgyan/HomogeneousModel
- name: wolframscript
  role: CLI runner for the RunHomogeneous.wl driver that executes the pole search
    over the $AnalysisConfig log-grid and writes homogeneous_params.wl/.json
  repo: https://github.com/ygevorgyan/HomogeneousModel
- name: Wolfram Engine
  role: Free runtime (version 12.0+) alternative to Mathematica for executing the
    package
- name: Mathematica 12.0+
  role: Interactive runtime alternative for running the package scripts
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202659861
    title: A user-friendly package and workflow for generating effective homogeneous
      rheologies for the study of the long-term orbital evolution of multilayered
      planetary bodies
  source_content_hash: sha256:a7c8003595de52112a79ceffc234007249dafeea4021df4e87fe9dcf5074fbfb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ygevorgyan/HomogeneousModel
  tools:
  - Mathematica
  - Mathematica 12.0+
  - Wolfram Engine
  - Wolfram Language
  - wolframscript
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# viscoelastic-relaxation-pole-identification

## Summary

Numerically identifies the secular relaxation poles and residues of the degree-2 tidal Love number k2(s) of a multilayered incompressible viscoelastic body via a propagator-matrix formulation, then inverts the response into an equivalent homogeneous generalized Voigt rheology usable by external orbital-evolution codes.

## When to use

Use when a layered planetary interior model (radii, densities, shear moduli, viscosities in ModelInput.wl) must be reduced to an effective homogeneous Voigt rheology for long-term tidal/orbital evolution calculations.

## When NOT to use

- The goal is a full time-domain viscoelastic relaxation spectrum solve rather than an equivalent homogeneous rheology for orbital-evolution codes
- The body model is a single homogeneous layer with no stratification, so no layered pole/residue identification is needed
- The input contains already at most one liquid run and the user only needs layer visualization (VisualizeLayers.wl suffices)

## Inputs

- ModelInput.wl configuration with $ModelConfig: layer radii, densities, shear moduli, viscosities, tidal frequency, optional layerNames
- $AnalysisConfig: workingPrecision, pole-search grid (gridLogMin/gridLogMax, gridPoints), frequency range, dominant-mode threshold
- Pre-configured test cases from test_cases/ (ModelInputMoon.wl 5-layer Moon, ModelInputEuropa.wl, ModelInputIo.wl, ModelInputEnceladus.wl)

## Outputs

- Love numbers (kE, kf, k2) and rheological parameters (gamma, alpha, eta)
- Secular relaxation modes (poles and residues) and Voigt elements, plus dominant-mode analysis
- output/homogeneous_params.wl and output/homogeneous_params.json (Voigt parameters for external tidal-evolution codes)
- Four comparison plots: real_k2_full.pdf, real_k2_reduced.pdf, imag_k2_full.pdf, imag_k2_reduced.pdf

## How to apply

Define the body in ModelInput.wl with $ModelConfig (rk, rhok, muk, etak ordered center-to-surface, liquid layers flagged by mu < 1 Pa) and $AnalysisConfig (workingPrecision, pole-search grid gridLogMin/gridLogMax, gridPoints, frequency range, dominant-mode threshold). Run wolframscript -file RunHomogeneous.wl from homogeneous_model/; the pipeline computes k2(s) via the 6x6 fundamental-matrix propagator (YMatrix.wl, NLayerJacobian.wl), searches the poles on the log-grid, and uses HomogeneousRheology.wl to invert residues into elastic, gravitational, viscous, and Voigt-element contributions. If FindRoot::brdig warnings appear, treat them as harmless; if fewer relaxation modes than expected are found, widen gridLogMin/gridLogMax or increase gridPoints and re-run. If the N-layer and Voigt curves of Re[k2] or -Im[k2] separate in the overlay plots, raise workingPrecision to 80 or 100 and recompute. Optionally apply dominant-mode selection over the orbital-evolution frequency range to discard negligible modes, yielding a reduced Voigt model with fewer relaxation elements that still preserves the tidal response.

## Related tools

- **Wolfram Language** (Language in which the package's propagator-matrix k2(s) computation, pole/residue search, and HomogeneousRheology inversion are implemented) — https://github.com/ygevorgyan/HomogeneousModel
- **wolframscript** (CLI runner for the RunHomogeneous.wl driver that executes the pole search over the $AnalysisConfig log-grid and writes homogeneous_params.wl/.json) — https://github.com/ygevorgyan/HomogeneousModel
- **Wolfram Engine** (Free runtime (version 12.0+) alternative to Mathematica for executing the package)
- **Mathematica 12.0+** (Interactive runtime alternative for running the package scripts)

## Examples

```
cp test_cases/ModelInputMoon.wl ModelInput.wl && wolframscript -file RunHomogeneous.wl
```

## Evaluation signals

- Overlay of Re[k2] and -Im[k2] curves: N-layer vs full Voigt and N-layer vs reduced Voigt must coincide with no visible separation over the analysis frequency range
- Number of identified secular relaxation modes matches expectation; if fewer, widen gridLogMin/gridLogMax or increase gridPoints and re-run
- output/ contains homogeneous_params.wl, homogeneous_params.json, and the four comparison plots; run completes in under 1 minute
- FindRoot::brdig warnings may appear but must not change results (harmless by documentation)
- Reduced Voigt model uses fewer relaxation elements than the full model while preserving the tidal response over the orbital-evolution frequency interval

## Limitations

- FindRoot::brdig warnings occur during the pole search; they are harmless but indicate the root finder hit bridge conditions
- Pole search depends on the log-grid: too narrow gridLogMin/gridLogMax or too few gridPoints can miss poles (fewer modes than expected)
- Numerical separation between N-layer and Voigt curves at default precision requires raising workingPrecision to 80 or 100
- The package targets incompressible multilayered bodies under tidal forcing; compressible rheologies are out of scope

## Evidence

- [abstract] numerical identification of the secular relaxation poles and residues of the layered model: "numerical identification of the secular relaxation poles and residues of the layered model"
- [readme] Pole search grid tuning: "If poles are missed (fewer modes than expected), widen
  gridLogMin/gridLogMax or increase gridPoints in $AnalysisConfig."
- [readme] Precision escalation: "If plots show separation between N-layer and full Voigt curves,
  increase workingPrecision to 80 or 100."
- [readme] Driver command: "1. Equivalent homogeneous body computation:

    wolframscript -file RunHomogeneous.wl"
- [intro] Liquid layer flagging: "Liquid layers are indicated by setting mu < 1 Pa."
