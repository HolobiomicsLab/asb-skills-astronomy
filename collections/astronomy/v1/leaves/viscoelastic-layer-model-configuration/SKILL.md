---
name: viscoelastic-layer-model-configuration
description: Use when defining or adapting a multilayered planetary interior model
  (radii, densities, shear moduli, viscosities, tidal frequency) for the HomogeneousModel
  package, e.g. specifying a stratified liquid core or selecting a test case.
when_to_use_negative:
- The input already contains at most one liquid run — the merge is a no-op and configuration
  can proceed without stratified-liquid considerations
- The target is a compressible or elastic-outer-shell body — the package handles multilayered
  incompressible bodies only
- Only output post-processing (plots, JSON/Wolfram Voigt parameters) is needed and
  ModelInput.wl is already configured
tools:
- name: wolframscript
  role: CLI front-end used to verify the Wolfram runtime and drive RunHomogeneous.wl
    / VisualizeLayers.wl with the configured ModelInput.wl
  repo: github.com/ygevorgyan/HomogeneousModel
- name: Wolfram Engine (free) or Mathematica 12.0+
  role: Required runtime for the HomogeneousModel package that consumes the configured
    layer model
  repo: github.com/ygevorgyan/HomogeneousModel
- name: HomogeneousRheology.wl
  role: Core module computing the equivalent homogeneous body from the configured
    layered model; accepts the "MergeAdjacentLiquids" -> False option
  repo: github.com/ygevorgyan/HomogeneousModel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202659861
    title: A user-friendly package and workflow for generating effective homogeneous
      rheologies for the study of the long-term orbital evolution of multilayered
      planetary bodies
  source_content_hash: sha256:db2606dd0305cc5603f26ad9cb31269dfc11c7f24d29cb1a229651e0404b2575
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HomogeneousRheology.wl
  - Wolfram Engine (free) or Mathematica 12.0+
  - wolframscript
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# viscoelastic-layer-model-configuration

## Summary

Configures the layered incompressible viscoelastic body definition in the Wolfram Language HomogeneousModel package by editing ModelInput.wl ($ModelConfig / $AnalysisConfig blocks) or copying a pre-configured Moon, Europa, Io, or Enceladus test case, as a prerequisite for computing Love numbers and an equivalent homogeneous generalized Voigt rheology.

## When to use

Use when defining or adapting a multilayered planetary interior model (radii, densities, shear moduli, viscosities, tidal frequency) for the HomogeneousModel package, e.g. specifying a stratified liquid core or selecting a test case.

## When NOT to use

- The input already contains at most one liquid run — the merge is a no-op and configuration can proceed without stratified-liquid considerations
- The target is a compressible or elastic-outer-shell body — the package handles multilayered incompressible bodies only
- Only output post-processing (plots, JSON/Wolfram Voigt parameters) is needed and ModelInput.wl is already configured

## Inputs

- ModelInput.wl configuration file with $ModelConfig (rk, rhok, muk, etak arrays, tidal frequency, optional layerNames) and $AnalysisConfig (precision, pole-search gridLogMin/gridLogMax/gridPoints, frequency range, dominant-mode threshold)
- Pre-configured test case files: test_cases/ModelInputMoon.wl (5-layer Moon), ModelInputEuropa.wl (4-layer Europa), ModelInputIo.wl (4-layer Io), ModelInputEnceladus.wl (3-layer Enceladus), ModelInputStratLiquid.wl (stratified-liquid demo)

## Outputs

- A valid ModelInput.wl defining an ordered multilayered incompressible body consumed by RunHomogeneous.wl and VisualizeLayers.wl
- Layer cross-section diagram (output/layer_cross_section.pdf) honouring optional layerNames labels

## How to apply

Edit ModelInput.wl inside homogeneous_model/, filling the $ModelConfig block (radii, densities, shear moduli, viscosities, tidal frequency, optional layer names) and the $AnalysisConfig block (precision, pole-search grid, frequency range, dominant-mode threshold). All layer arrays (rk, rhok, muk, etak) must have equal length and be ordered from center (layer 1) to surface (layer N); liquid layers are indicated by setting mu < 1 Pa. If consecutive liquid sub-layers are supplied (e.g. a stratified core), the pipeline automatically collapses them into one equivalent liquid layer using rho_eff = sum_k rho_k (r_k^3 - r_{k-1}^3)/(r_outer^3 - r_inner^3), which preserves total mass; this merge can be disabled via "MergeAdjacentLiquids" -> False passed to computeHomogeneous. Rather than writing a configuration from scratch, a pre-configured test case can be copied over ModelInput.wl (cp test_cases/ModelInputEuropa.wl ModelInput.wl). Optionally provide a "layerNames" key (e.g. {"Inner core", "Outer core", ...}) to label the layer cross-section visualization; otherwise layers are generically labelled "Layer i (solid/liquid)".

## Related tools

- **wolframscript** (CLI front-end used to verify the Wolfram runtime and drive RunHomogeneous.wl / VisualizeLayers.wl with the configured ModelInput.wl) — github.com/ygevorgyan/HomogeneousModel
- **Wolfram Engine (free) or Mathematica 12.0+** (Required runtime for the HomogeneousModel package that consumes the configured layer model) — github.com/ygevorgyan/HomogeneousModel
- **HomogeneousRheology.wl** (Core module computing the equivalent homogeneous body from the configured layered model; accepts the "MergeAdjacentLiquids" -> False option) — github.com/ygevorgyan/HomogeneousModel

## Examples

```
cp test_cases/ModelInputStratLiquid.wl ModelInput.wl
```

## Evaluation signals

- All layer arrays (rk, rhok, muk, etak) have identical length and are ordered from center (layer 1) to surface (layer N)
- Every intended liquid layer has mu < 1 Pa so the pipeline classifies it as liquid
- For stratified liquid runs, the merged region's volume-weighted mean density rho_eff preserves the total mass of the merged region exactly; reductions from sub-layered and pre-merged bodies agree to all printed digits
- wolframscript -version succeeds (Wolfram Engine or Mathematica 12.0+ present) and commands are run from the homogeneous_model/ directory
- If poles are missed (fewer modes than expected), widen gridLogMin/gridLogMax or increase gridPoints in $AnalysisConfig

## Limitations

- FindRoot::brdig warnings are harmless and may be ignored
- If plots show separation between N-layer and full Voigt curves, increase workingPrecision to 80 or 100
- Requires Wolfram Engine (free) or Mathematica version 12.0+; if wolframscript is not on PATH it must be called directly (e.g. /Applications/Mathematica.app/Contents/MacOS/wolframscript)

## Evidence

- [intro] All layer arrays (rk, rhok, muk, etak) must have the same length and are ordered from center (layer 1) to surface (layer N). Liquid layers are indicated by setting mu < 1 Pa.: "All layer arrays (rk, rhok, muk, etak) must have the same length and are ordered from center (layer 1) to surface (layer N)."
- [readme] $ModelConfig block contents: "$ModelConfig      radii, densities, shear moduli, viscosities,
                    tidal frequency, optional layer names"
- [readme] Merge rationale for stratified liquid sub-layers: "the pipeline collapses them automatically into a single equivalent
liquid layer ... which preserves the total mass of the merged region"
- [methods] Test-case copy workflow: "To use one, copy it over ModelInput.wl:

    cp test_cases/ModelInputEuropa.wl ModelInput.wl"
- [readme] layerNames labelling option: "An optional "layerNames" key can provide custom labels for the visualization (e.g., {"Inner core", "Outer core", ...})."
- [intro] Runtime prerequisite: "Wolfram Engine (free) or Mathematica, version 12.0+."
