---
name: lunar-layered-interior-modeling
description: Use when a multilayered lunar (or other planetary) interior model with
  radii, densities, shear moduli, and viscosities must be reduced to an equivalent
  homogeneous generalized Voigt body for long-term tidal/orbital evolution calculations.
when_to_use_negative:
- The goal is a forward-only Love-number calculation without an equivalent homogeneous
  rheology — the pipeline's purpose is the inversion into a generalized Voigt body,
  not standalone forward modeling
- The body is compressible — the package targets multilayered incompressible bodies
  under tidal forcing
- The tidal response must be resolved over frequencies outside the $AnalysisConfig
  frequency range/pole-search grid without re-tuning gridLogMin/gridLogMax and gridPoints
edam_operation: http://edamontology.org/operation_3799
tools:
- name: Wolfram Language
  role: Open-source package language implementing the propagator-matrix forward Love-number
    computation, pole/residue identification, and inversion into the equivalent homogeneous
    generalized Voigt rheology
  repo: github.com/ygevorgyan/HomogeneousModel
- name: wolframscript
  role: CLI driver used to execute RunHomogeneous.wl for the lunar five-layer run
  repo: github.com/ygevorgyan/HomogeneousModel
- name: Mathematica
  role: Alternative (version 12.0+) to the free Wolfram Engine for running the package
- name: Wolfram Engine
  role: Free runtime (version 12.0+) for executing the RunHomogeneous.wl driver
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202659861
    title: A user-friendly package and workflow for generating effective homogeneous
      rheologies for the study of the long-term orbital evolution of multilayered
      planetary bodies
  source_content_hash: sha256:0003685bf52d66854eae31a30807098dbb185e1d61b072554593c5ae4e742bdc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mathematica
  - Wolfram Engine
  - Wolfram Language
  - wolframscript
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lunar-layered-interior-modeling

## Summary

Apply the Wolfram Language HomogeneousModel package to a five-layer lunar interior configuration to compute degree-2 tidal Love numbers and invert the layered response into an equivalent homogeneous generalized Voigt rheology usable by external tidal-evolution codes.

## When to use

Use when a multilayered lunar (or other planetary) interior model with radii, densities, shear moduli, and viscosities must be reduced to an equivalent homogeneous generalized Voigt body for long-term tidal/orbital evolution calculations.

## When NOT to use

- The goal is a forward-only Love-number calculation without an equivalent homogeneous rheology — the pipeline's purpose is the inversion into a generalized Voigt body, not standalone forward modeling
- The body is compressible — the package targets multilayered incompressible bodies under tidal forcing
- The tidal response must be resolved over frequencies outside the $AnalysisConfig frequency range/pole-search grid without re-tuning gridLogMin/gridLogMax and gridPoints

## Inputs

- ModelInput.wl configuration defining $ModelConfig (radii rk, densities rhok, shear moduli muk, viscosities etak, tidal frequency, optional layerNames) and $AnalysisConfig (workingPrecision, pole-search grid gridLogMin/gridLogMax/gridPoints, frequency range, dominant-mode threshold)
- Five-layer lunar test case test_cases/ModelInputMoon.wl
- Wolfram Engine (free) or Mathematica version 12.0+ with wolframscript on PATH

## Outputs

- Love numbers kE, kf, k2 and rheological parameters gamma, alpha, eta printed as terminal tables
- Secular relaxation modes, Voigt elements, and dominant-mode analysis
- output/homogeneous_params.wl and output/homogeneous_params.json (Voigt parameters in Wolfram and JSON formats)
- Four comparison plots: output/real_k2_full.pdf, output/real_k2_reduced.pdf, output/imag_k2_full.pdf, output/imag_k2_reduced.pdf

## How to apply

Copy the pre-configured five-layer lunar case over the user configuration (cp test_cases/ModelInputMoon.wl ModelInput.wl); $ModelConfig defines radii, densities, shear moduli, viscosities, tidal frequency, and optional layer names ordered center-to-surface, with liquid layers indicated by mu < 1 Pa. From the homogeneous_model/ directory run 'wolframscript -file RunHomogeneous.wl', which performs the forward propagator-matrix computation of k2(s), a pole search over the log-grid in $AnalysisConfig (gridLogMin/gridLogMax, gridPoints), and inversion via the HomogeneousRheology module into the equivalent homogeneous generalized Voigt body; consecutive liquid layers are merged by default into one equivalent layer with volume-weighted mean density rho_eff = sum_k rho_k (r_k^3 - r_{k-1}^3)/(r_outer^3 - r_inner^3). Treat FindRoot::brdig warnings as harmless; if fewer secular relaxation modes than expected are found, widen gridLogMin/gridLogMax or increase gridPoints, and if the Re[k2]/-Im[k2] overlay plots show separation between N-layer and full Voigt curves, raise workingPrecision to 80 or 100 and re-run. Confirm the run finishes in under 1 minute and collects the outputs: Love numbers (kE, kf, k2), rheological parameters (gamma, alpha, eta), secular relaxation modes, Voigt elements, dominant-mode analysis, four comparison plots, and homogeneous_params.wl / homogeneous_params.json in output/.

## Related tools

- **Wolfram Language** (Open-source package language implementing the propagator-matrix forward Love-number computation, pole/residue identification, and inversion into the equivalent homogeneous generalized Voigt rheology) — github.com/ygevorgyan/HomogeneousModel
- **wolframscript** (CLI driver used to execute RunHomogeneous.wl for the lunar five-layer run) — github.com/ygevorgyan/HomogeneousModel
- **Mathematica** (Alternative (version 12.0+) to the free Wolfram Engine for running the package)
- **Wolfram Engine** (Free runtime (version 12.0+) for executing the RunHomogeneous.wl driver)

## Examples

```
cp test_cases/ModelInputMoon.wl ModelInput.wl && wolframscript -file RunHomogeneous.wl
```

## Evaluation signals

- Run completes in under 1 minute and all listed outputs (Love numbers, gamma/alpha/eta, secular modes, Voigt elements, four plots, homogeneous_params.wl/.json) appear in output/
- Overlay plots of Re[k2] and -Im[k2] show the N-layer, full Voigt, and reduced Voigt curves coinciding; separation between N-layer and full Voigt curves indicates insufficient workingPrecision (raise to 80 or 100)
- Number of identified secular relaxation modes matches expectations; a shortfall means poles were missed and gridLogMin/gridLogMax/gridPoints in $AnalysisConfig must be widened/increased
- FindRoot::brdig warnings are emitted but are harmless and do not invalidate the run
- homogeneous_params.wl and homogeneous_params.json parameterize the equivalent body by elastic, gravitational, viscous, and Voigt-element contributions in a format loadable by external tidal-evolution codes

## Limitations

- FindRoot::brdig warnings occur during the pole search; they are harmless but signal the numerical root-finding is bridge-constrained
- Pole search can miss relaxation modes if gridLogMin/gridLogMax is too narrow or gridPoints too small
- Plot separation between N-layer and full Voigt curves at default workingPrecision requires raising workingPrecision to 80 or 100
- The model assumes incompressible stratified viscoelastic layers; liquid layers must be encoded with mu < 1 Pa

## Evidence

- [methods] five-layer lunar configuration: "we apply the package to a five-layer lunar interior model and obtain its equivalent generalized Voigt representation, together with a reduced model that preserves the tidal response over the"
- [methods] driver invocation: "wolframscript -file RunHomogeneous.wl"
- [readme] configuration blocks: "Edit ModelInput.wl to define the body.  It contains two blocks:

  $ModelConfig      radii, densities, shear moduli, viscosities,
                    tidal frequency, optional layer names"
- [readme] liquid-layer convention: "Liquid layers are indicated by setting mu < 1 Pa."
- [readme] troubleshooting pole search: "If poles are missed (fewer modes than expected), widen
  gridLogMin/gridLogMax or increase gridPoints in $AnalysisConfig."
- [readme] precision fallback: "If plots show separation between N-layer and full Voigt curves,
  increase workingPrecision to 80 or 100."
- [readme] runtime and outputs: "Outputs Love numbers (kE, kf, k2), rheological parameters
   (gamma, alpha, eta), secular relaxation modes, Voigt elements,
   dominant mode analysis, and four comparison plots.
   Runtime: under 1"
- [readme] harmless warnings: "FindRoot::brdig warnings are harmless."
