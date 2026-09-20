---
name: dominant-mode-selection-and-rheology-reduction
description: Use when a layered viscoelastic body's full Voigt representation has
  more relaxation elements than needed and the tidal response only matters over a
  specified frequency interval (e.g. the orbital-evolution range), requiring a reduced
  model.
when_to_use_negative:
- The tidal response must be accurate outside the prescribed analysis frequency range
  — the reduced model is only guaranteed to preserve k2 over the $AnalysisConfig frequency
  interval
- The pole-search grid was too narrow (fewer modes than expected), since the dominant-mode
  selection would then operate on an incomplete set of secular relaxation modes
- You need the full Voigt representation with all relaxation elements rather than
  a reduced model (compare with real_k2_full.pdf / imag_k2_full.pdf instead)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3315
tools:
- name: Wolfram Language
  role: Language of the open-source package that implements the dominant-mode selection
    and constructs the effective homogeneous generalized Voigt rheology
  repo: https://github.com/ygevorgyan/HomogeneousModel
- name: wolframscript
  role: Command-line driver used to execute RunHomogeneous.wl, which performs the
    pole search, dominant-mode analysis, and reduced-model comparison
  repo: https://github.com/ygevorgyan/HomogeneousModel
- name: Mathematica 12.0+
  role: Alternative (paid) runtime to the free Wolfram Engine for executing the package
    scripts
- name: Wolfram Engine
  role: Free runtime option (version 12.0+) for running the package's wolframscript
    drivers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202659861
    title: A user-friendly package and workflow for generating effective homogeneous
      rheologies for the study of the long-term orbital evolution of multilayered
      planetary bodies
  source_content_hash: sha256:7c975c25b5493ee81086162f43d5ced8f441e96f03167059ece6920b6d0ef5f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ygevorgyan/HomogeneousModel
  tools:
  - Mathematica 12.0+
  - Wolfram Engine
  - Wolfram Language
  - wolframscript
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# dominant-mode-selection-and-rheology-reduction

## Summary

Applies the package's optional dominant-mode selection procedure to discard negligible secular relaxation modes from an equivalent homogeneous generalized Voigt model, producing a reduced rheology with fewer relaxation elements that preserves the tidal Love number k2 response over a prescribed frequency range.

## When to use

Use when a layered viscoelastic body's full Voigt representation has more relaxation elements than needed and the tidal response only matters over a specified frequency interval (e.g. the orbital-evolution range), requiring a reduced model.

## When NOT to use

- The tidal response must be accurate outside the prescribed analysis frequency range — the reduced model is only guaranteed to preserve k2 over the $AnalysisConfig frequency interval
- The pole-search grid was too narrow (fewer modes than expected), since the dominant-mode selection would then operate on an incomplete set of secular relaxation modes
- You need the full Voigt representation with all relaxation elements rather than a reduced model (compare with real_k2_full.pdf / imag_k2_full.pdf instead)

## Inputs

- ModelInput.wl configuration with $ModelConfig (radii rk, densities rhok, shear moduli muk, viscosities etak, tidal frequency, optional layerNames) and $AnalysisConfig (workingPrecision, pole-search grid gridLogMin/gridLogMax/gridPoints, frequency range, dominant-mode threshold)
- Equivalent homogeneous generalized Voigt representation of the layered body (secular relaxation poles and residues, Voigt elements)
- Pre-configured test case such as test_cases/ModelInputMoon.wl (five-layer Moon)

## Outputs

- Reduced Voigt model with fewer relaxation elements preserving the tidal response over the analysis frequency range
- output/real_k2_reduced.pdf — Re[k2]: N-layer model versus reduced Voigt model
- output/imag_k2_reduced.pdf — -Im[k2]: N-layer model versus reduced Voigt model
- Dominant-mode analysis table and Voigt parameters (output/homogeneous_params.wl, output/homogeneous_params.json)

## How to apply

First run the forward pipeline: copy the pre-configured test case (e.g. test_cases/ModelInputMoon.wl, the five-layer Moon) over ModelInput.wl, then run 'wolframscript -file RunHomogeneous.wl' from homogeneous_model/ to compute the N-layer degree-2 Love number k2(s) via the propagator-matrix formulation and invert the response into the equivalent homogeneous generalized Voigt body. Set the pole-search grid (gridLogMin/gridLogMax, gridPoints) in $AnalysisConfig wide enough that no secular relaxation poles are missed. Then apply the dominant-mode selection procedure using the dominant-mode threshold and analysis frequency range in $AnalysisConfig to discard negligible relaxation modes and obtain the reduced Voigt model with fewer relaxation elements. Compare Re[k2] and -Im[k2] of the N-layer model against the reduced Voigt model over the prescribed frequency range; if the curves separate, raise workingPrecision to 80 or 100 and recompute before judging the reduction valid.

## Related tools

- **Wolfram Language** (Language of the open-source package that implements the dominant-mode selection and constructs the effective homogeneous generalized Voigt rheology) — https://github.com/ygevorgyan/HomogeneousModel
- **wolframscript** (Command-line driver used to execute RunHomogeneous.wl, which performs the pole search, dominant-mode analysis, and reduced-model comparison) — https://github.com/ygevorgyan/HomogeneousModel
- **Mathematica 12.0+** (Alternative (paid) runtime to the free Wolfram Engine for executing the package scripts)
- **Wolfram Engine** (Free runtime option (version 12.0+) for running the package's wolframscript drivers)

## Examples

```
cp test_cases/ModelInputMoon.wl ModelInput.wl && wolframscript -file RunHomogeneous.wl
```

## Evaluation signals

- The reduced Voigt curve must overlay the N-layer curve in both output/real_k2_reduced.pdf and output/imag_k2_reduced.pdf over the analysis frequency range, with no visible separation
- The reduced model uses fewer relaxation elements (Voigt elements) than the full equivalent generalized Voigt representation
- The pole-search must yield the expected number of secular relaxation modes — if fewer modes than expected are found, widen gridLogMin/gridLogMax or increase gridPoints in $AnalysisConfig and rerun
- FindRoot::brdig warnings emitted during the pole search are harmless and do not invalidate the run
- If N-layer and reduced Voigt curves separate in the plots, recompute with workingPrecision raised to 80 or 100; only then is the comparison conclusive

## Limitations

- FindRoot::brdig warnings occur during the pole search; they are harmless but can look like failures
- If poles are missed (fewer modes than expected), the gridLogMin/gridLogMax range or gridPoints in $AnalysisConfig must be widened
- Numerical precision issues can cause visible separation between N-layer and Voigt curves; workingPrecision must be raised to 80 or 100
- The reduction is valid only over the prescribed frequency range (the interval relevant for orbital evolution); outside it the reduced model's response is not guaranteed to match the layered model

## Evidence

- [abstract] we apply the package to a five-layer lunar interior model and obtain its equivalent generalized Voigt representation, together with a reduced model that preserves the tidal response over the: "we apply the package to a five-layer lunar interior model and obtain its equivalent generalized Voigt representation, together with a reduced model that preserves the tidal response over the"
- [abstract] optional dominant-mode selection procedure: "includes an optional dominant-mode selection procedure for obtaining reduced rheological models over a prescribed frequency range"
- [readme] $AnalysisConfig precision, pole-search grid, frequency range, dominant-mode threshold: "$AnalysisConfig   precision, pole-search grid, frequency range,
                    dominant-mode threshold"
- [readme] If poles are missed, widen gridLogMin/gridLogMax or increase gridPoints: "If poles are missed (fewer modes than expected), widen
  gridLogMin/gridLogMax or increase gridPoints in $AnalysisConfig."
- [readme] If plots show separation between N-layer and full Voigt curves, increase workingPrecision to 80 or 100: "If plots show separation between N-layer and full Voigt curves,
  increase workingPrecision to 80 or 100."
