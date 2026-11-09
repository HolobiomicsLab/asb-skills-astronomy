---
name: generalized-voigt-parameter-fitting
description: Use when a layered planetary interior model (radii, densities, shear
  moduli, viscosities) must be replaced by an effective homogeneous Voigt rheology
  for long-term orbital/spin evolution, or when a reduced model over a prescribed
  frequency range is needed.
when_to_use_negative:
- The problem requires solving the full layered boundary-value problem at each step
  of an orbital calculation — the package exists precisely to avoid that by providing
  precomputed effective parameters
- The body or rheology is not incompressible/stratified viscoelastic with degree-2
  tidal forcing — the package is restricted to the incompressible propagator-matrix
  formulation
- Only a layer cross-section visualization is needed (use VisualizeLayers.wl) rather
  than Love numbers and rheological parameter inversion
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Wolfram Language
  role: Language of the open-source package that constructs the effective homogeneous
    generalized Voigt rheology
  repo: github.com/ygevorgyan/HomogeneousModel
- name: wolframscript
  role: CLI driver used to run RunHomogeneous.wl and verify installation (wolframscript
    -version)
- name: HomogeneousRheology.wl
  role: Core src/ module performing the equivalent homogeneous body computation
  repo: github.com/ygevorgyan/HomogeneousModel
- name: NLayerJacobian.wl
  role: Core src/ module computing the N-layer propagator and k2(s)
  repo: github.com/ygevorgyan/HomogeneousModel
- name: Wolfram Engine / Mathematica 12.0+
  role: Runtime prerequisite for executing the package
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202659861
    title: A user-friendly package and workflow for generating effective homogeneous
      rheologies for the study of the long-term orbital evolution of multilayered
      planetary bodies
  source_content_hash: sha256:ca56324c4b9ab09de7829d2e5d5844f18945f4caf8af5c363c4880de4d3d4657
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HomogeneousRheology.wl
  - NLayerJacobian.wl
  - Wolfram Engine / Mathematica 12.0+
  - Wolfram Language
  - wolframscript
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# generalized-voigt-parameter-fitting

## Summary

Use the HomogeneousModel Wolfram Language package to invert the degree-2 tidal Love number response k2(s) of a multilayered incompressible planetary body into an equivalent homogeneous generalized Voigt rheology, and optionally reduce it to a dominant-mode model that preserves the tidal response with fewer relaxation elements.

## When to use

Use when a layered planetary interior model (radii, densities, shear moduli, viscosities) must be replaced by an effective homogeneous Voigt rheology for long-term orbital/spin evolution, or when a reduced model over a prescribed frequency range is needed.

## When NOT to use

- The problem requires solving the full layered boundary-value problem at each step of an orbital calculation — the package exists precisely to avoid that by providing precomputed effective parameters
- The body or rheology is not incompressible/stratified viscoelastic with degree-2 tidal forcing — the package is restricted to the incompressible propagator-matrix formulation
- Only a layer cross-section visualization is needed (use VisualizeLayers.wl) rather than Love numbers and rheological parameter inversion

## Inputs

- ModelInput.wl configuration with $ModelConfig (rk, rhok, muk, etak arrays ordered from center to surface; liquid layers indicated by mu < 1 Pa) and $AnalysisConfig (workingPrecision, gridLogMin/gridLogMax, gridPoints, frequency range, dominant-mode threshold)
- Pre-configured test cases from test_cases/ (ModelInputMoon.wl 5-layer Moon, ModelInputEuropa.wl, ModelInputIo.wl, ModelInputEnceladus.wl)

## Outputs

- homogeneous_params.wl and homogeneous_params.json — Voigt rheological parameters (elastic, gravitational, viscous, Voigt-element contributions)
- output/real_k2_reduced.pdf and output/imag_k2_reduced.pdf — Re[k2] and -Im[k2] of N-layer vs reduced Voigt model
- output/real_k2_full.pdf and output/imag_k2_full.pdf — N-layer vs full Voigt comparison plots
- Terminal tables of Love numbers (kE, kf, k2), rheological parameters (gamma, alpha, eta), secular relaxation modes, Voigt elements, and dominant mode analysis

## How to apply

Define the body in ModelInput.wl via $ModelConfig (radii, densities, shear moduli, viscosities, tidal frequency) and $AnalysisConfig (precision, pole-search grid gridLogMin/gridLogMax, gridPoints, frequency range, dominant-mode threshold), with layers ordered center-to-surface and liquid layers flagged by mu < 1 Pa. Run RunHomogeneous.wl from homogeneous_model/ with wolframscript, which computes k2(s) via the propagator-matrix formulation, identifies the secular relaxation poles and residues, and inverts the response into the compliance of an equivalent homogeneous generalized Voigt body. Apply the optional dominant-mode selection procedure over the prescribed frequency interval to discard negligible relaxation modes and obtain a reduced Voigt model with fewer relaxation elements. If the N-layer and Voigt curves separate, raise workingPrecision to 80 or 100; if poles are missed, widen gridLogMin/gridLogMax or increase gridPoints. Validate by checking that the reduced Voigt curve overlays the N-layer curve in the real_k2_reduced.pdf and imag_k2_reduced.pdf comparison plots, and consume homogeneous_params.wl or homogeneous_params.json as rheological parameters for external tidal-evolution codes.

## Related tools

- **Wolfram Language** (Language of the open-source package that constructs the effective homogeneous generalized Voigt rheology) — github.com/ygevorgyan/HomogeneousModel
- **wolframscript** (CLI driver used to run RunHomogeneous.wl and verify installation (wolframscript -version))
- **HomogeneousRheology.wl** (Core src/ module performing the equivalent homogeneous body computation) — github.com/ygevorgyan/HomogeneousModel
- **NLayerJacobian.wl** (Core src/ module computing the N-layer propagator and k2(s)) — github.com/ygevorgyan/HomogeneousModel
- **Wolfram Engine / Mathematica 12.0+** (Runtime prerequisite for executing the package)

## Examples

```
cp test_cases/ModelInputMoon.wl ModelInput.wl; wolframscript -file RunHomogeneous.wl
```

## Evaluation signals

- The reduced Voigt curve must overlay the N-layer curve in both output/real_k2_reduced.pdf and output/imag_k2_reduced.pdf over the analysis frequency range with no visible separation
- The dominant-mode reduction must use fewer relaxation elements than the full Voigt representation while preserving the tidal response over the orbital-evolution frequency interval
- The stratified-liquid merge (volume-weighted rho_eff preserving total mass exactly) must agree with the already-merged equivalent body to all printed digits
- Pole identification completeness: if fewer modes than expected are found, widen gridLogMin/gridLogMax or increase gridPoints in $AnalysisConfig and rerun
- FindRoot::brdig warnings during the pole search are expected and do not invalidate the run

## Limitations

- FindRoot::brdig warnings during the pole search are harmless but may alarm users
- If poles are missed (fewer modes than expected), the pole-search grid (gridLogMin/gridLogMax, gridPoints) must be widened in $AnalysisConfig
- If plots show separation between N-layer and full Voigt curves, workingPrecision must be increased to 80 or 100
- The model is for incompressible stratified viscoelastic bodies under degree-2 tidal forcing; a run completes in under 1 minute but was demonstrated on small-body test cases (Moon, Europa, Io, Enceladus)

## Evidence

- [abstract] Forward computation plus inversion pipeline: [evidence span withheld — anchor status: not_found]
- [abstract] Reduced model case study: "we apply the package to a five-layer lunar interior model and obtain its equivalent generalized Voigt representation, together with a reduced model that preserves the tidal response over the"
- [readme] Main driver invocation: "Equivalent homogeneous body computation:

    wolframscript -file RunHomogeneous.wl"
- [readme] Precision escalation rule: "If plots show separation between N-layer and full Voigt curves,
  increase workingPrecision to 80 or 100."
- [readme] Pole-search completeness rule: "If poles are missed (fewer modes than expected), widen
  gridLogMin/gridLogMax or increase gridPoints in $AnalysisConfig."
