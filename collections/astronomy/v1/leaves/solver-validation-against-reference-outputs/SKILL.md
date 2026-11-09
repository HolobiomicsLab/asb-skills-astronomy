---
name: solver-validation-against-reference-outputs
description: Use when a re-implemented level-population or line-emissivity solver
  must be validated against published HyLight reference values (3p population 1.64e-17
  cm**(-3); H alpha emissivity 3.451e-21 erg/(cm**3*s)) at ne=1e2 cm**(-3), nHI=1e-5
  cm**(-3), nHII=1e2 cm**(-3), T=1e4 K.
when_to_use_negative:
- The gas conditions or atomic-model settings (Case A, different nmax, no recombination/collision
  terms) differ from the reference configuration, so comparison to the 1e4 K / 100
  cm**(-3) reference values would be invalid.
- The goal is emissivity benchmarking against Cloudy for Balmer, Paschen, and Brackett
  series rather than validating a standalone reconstructed solver.
- Non-equilibrium thermochemistry or radiation-hydrodynamical outputs are the target,
  since the reference values apply to steady-state, typical photoionised nebular conditions.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: HyLight
  role: Atomic model providing the reference 3p level population and H alpha emissivity
    under reference nebular conditions
  repo: https://github.com/YuankangLiu/HyLight
- name: hylightpy
  role: Python package (version 0.0.21) exposing HIAtom, compute_level_pop, and get_line_emissivity
    used to generate reference outputs
  repo: https://github.com/YuankangLiu/HyLight
- name: unyt
  role: Specifies gas density and temperature inputs with physical units (cm**(-3),
    K)
- name: numpy
  role: Numerical backend for the reconstructed rate-equation solve
- name: Python
  role: Implementation language for both the reconstructed solver and the validation
    script
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stag429
    title: The <scp>HyLight</scp> model for hydrogen emission lines in simulated nebulae
  source_content_hash: sha256:0d292b5e124d3408da8ac35a33e3fe043cf846da04ebfdb8bb0e4b59a7c8b76c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuankangLiu/HyLight
  tools:
  - HyLight
  - Python
  - hylightpy
  - numpy
  - unyt
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# solver-validation-against-reference-outputs

## Summary

Validates a reconstructed steady-state Case B hydrogen level-population solver by comparing its 3p level population and H alpha (n=3 to n=2) emissivity against HyLight reference-condition outputs, with a ~1 per cent agreement acceptance criterion.

## When to use

Use when a re-implemented level-population or line-emissivity solver must be validated against published HyLight reference values (3p population 1.64e-17 cm**(-3); H alpha emissivity 3.451e-21 erg/(cm**3*s)) at ne=1e2 cm**(-3), nHI=1e-5 cm**(-3), nHII=1e2 cm**(-3), T=1e4 K.

## When NOT to use

- The gas conditions or atomic-model settings (Case A, different nmax, no recombination/collision terms) differ from the reference configuration, so comparison to the 1e4 K / 100 cm**(-3) reference values would be invalid.
- The goal is emissivity benchmarking against Cloudy for Balmer, Paschen, and Brackett series rather than validating a standalone reconstructed solver.
- Non-equilibrium thermochemistry or radiation-hydrodynamical outputs are the target, since the reference values apply to steady-state, typical photoionised nebular conditions.

## Inputs

- Gas conditions as unyt arrays: ne = 1e2 cm**(-3), nHI = 1e-5 cm**(-3), nHII = 1e2 cm**(-3), temp = 1e4 K
- HIAtom configuration: nmax = 50 levels, caseB=True, recom=True, coll=True, cache_path='./'
- Reference values: 3p level population 1.64e-17 cm**(-3) and H alpha emissivity 3.451e-21 erg/(cm**3*s)

## Outputs

- Solved 3p (n=3, l=1) level population density in cm**(-3)
- H alpha (n=3 to n=2) line emissivity in erg/(cm**3*s)
- solver_validation_results.csv containing reconstructed vs. reference values

## How to apply

Construct the rate-equation system for atomic hydrogen up to nmax=50 levels in Case B, including direct recombination cascade terms and collisional excitation from the ground state. Specify gas conditions with the unyt package (ne = 1e2 cm**(-3), nHI = 1e-5 cm**(-3), nHII = 1e2 cm**(-3), temp = 1e4 K) and solve the steady-state population equations using hylightpy.HIAtom configured with caseB=True, recom=True, coll=True. Evaluate the 3p (n=3, l=1) population via compute_level_pop and the H alpha emissivity via get_line_emissivity(nupper=3, nlower=2). Compare both reconstructed outputs to the HyLight references: 3p population 1.64e-17 cm**(-3) and H alpha emissivity 3.451e-21 erg/(cm**3*s). The acceptance criterion is agreement within approximately 1 per cent, consistent with HyLight's reported accuracy versus Cloudy under typical photoionised nebular conditions; write both validated values to solver_validation_results.csv.

## Related tools

- **HyLight** (Atomic model providing the reference 3p level population and H alpha emissivity under reference nebular conditions) — https://github.com/YuankangLiu/HyLight
- **hylightpy** (Python package (version 0.0.21) exposing HIAtom, compute_level_pop, and get_line_emissivity used to generate reference outputs) — https://github.com/YuankangLiu/HyLight
- **unyt** (Specifies gas density and temperature inputs with physical units (cm**(-3), K))
- **numpy** (Numerical backend for the reconstructed rate-equation solve)
- **Python** (Implementation language for both the reconstructed solver and the validation script)

## Examples

```
HI = hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'); levelpop = HI.compute_level_pop(nHII=nHII, ne=ne, nHI=nHI, temp=temp, n=3, l=1)
```

## Evaluation signals

- Reconstructed 3p (n=3, l=1) population matches 1.64e-17 cm**(-3) within ~1 per cent.
- Reconstructed H alpha emissivity matches 3.451e-21 erg/(cm**3*s) within ~1 per cent.
- solver_validation_results.csv is written and contains both reconstructed values alongside the references.
- HIAtom was initialised with nmax=50, caseB=True, recom=True, coll=True, matching the documented configuration.
- Inputs were passed as unyt arrays with correct units (cm**(-3), K), matching the reference-condition specification.

## Limitations

- The ~1 per cent agreement holds only under typical photoionised nebular conditions; discrepancies of several tens of per cent arise relative to other published calculations.
- Validation is tied to the reference condition (ne=100 cm**(-3), nHI=1e-5 cm**(-3), nHII=100 cm**(-3), T=1e4 K); other conditions require new reference values.
- No version number is stated for unyt in the source; hylightpy version is 0.0.21.

## Evidence

- [methods] Level population of 3p state is 1.64e-17 cm**(-3): "Level population of 3p state is 1.64e-17 cm**(-3)"
- [methods] H alpha reference emissivity: "Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s)"
- [abstract] 1 per cent accuracy claim: "HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent"
- [readme] compute_level_pop description (readme): [evidence span withheld — anchor status: external_doc]
- [methods] unyt usage: "We utilise ``unyt`` package to specify the gas density and temperature."
- [readme] HIAtom initialisation (readme): "HI = hylightpy.HIAtom(nmax = 50, verbose=True, caseB=True, 
                      recom=True, coll=True, 
                      cache_path='./')"
