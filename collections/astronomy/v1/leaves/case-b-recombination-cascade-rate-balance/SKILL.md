---
name: case-b-recombination-cascade-rate-balance
description: Use when you need hydrogen excited-state level population densities or
  recombination line emissivities (e.g. H alpha n=3->2) for photoionised gas with
  known ne, nHI, nHII, and temperature, under Case B assumptions.
when_to_use_negative:
- The gas requires non-equilibrium thermochemistry or full radiative-transfer coupling
  rather than steady-state Case B populations — HyLight assumes steady state under
  photoionised nebular conditions
- You need Line ratios involving the ground state (Case A) — the skill constructs
  Case B with caseB=True, which depopulates via Lyman-continuum escape
- The target species is not atomic hydrogen — the HIAtom rate system covers only hydrogen
  levels up to nmax
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: HyLight
  role: Python atomic model whose HIAtom class solves the Case B level-population
    rate equations and returns line emissivities
  repo: https://github.com/YuankangLiu/HyLight
- name: hylightpy
  role: PyPI package providing HIAtom, compute_level_pop and get_line_emissivity (version
    0.0.21 used in the paper)
  repo: https://github.com/YuankangLiu/HyLight
- name: unyt
  role: Specifies gas density and temperature arrays with units (cm**(-3), K) passed
    to compute_level_pop and get_line_emissivity
- name: numpy
  role: Array support used in the HyLight example workflow
- name: Cloudy
  role: 'Benchmark reference: HyLight reproduces Cloudy Balmer, Paschen and Brackett
    emissivities to within 1 per cent'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stag429
    title: The <scp>HyLight</scp> model for hydrogen emission lines in simulated nebulae
  source_content_hash: sha256:44b5bed0b4b47405e9d733391c18190f3348b34869b5f8488f342531dd2736c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuankangLiu/HyLight
  tools:
  - Cloudy
  - HyLight
  - Python
  - hylightpy
  - numpy
  - unyt
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# case-b-recombination-cascade-rate-balance

## Summary

Solves the steady-state Case B rate-equation system for excited-state populations of atomic hydrogen (up to nmax=50 levels) with direct recombination cascades and collisional excitation from the ground state, using HyLight's HIAtom.compute_level_pop, and validates against reference values (3p population 1.64e-17 cm**(-3); H alpha emissivity 3.451e-21 erg/(cm**3*s)) at ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, T=1e4 K.

## When to use

Use when you need hydrogen excited-state level population densities or recombination line emissivities (e.g. H alpha n=3->2) for photoionised gas with known ne, nHI, nHII, and temperature, under Case B assumptions.

## When NOT to use

- The gas requires non-equilibrium thermochemistry or full radiative-transfer coupling rather than steady-state Case B populations — HyLight assumes steady state under photoionised nebular conditions
- You need Line ratios involving the ground state (Case A) — the skill constructs Case B with caseB=True, which depopulates via Lyman-continuum escape
- The target species is not atomic hydrogen — the HIAtom rate system covers only hydrogen levels up to nmax

## Inputs

- Electron density ne as unyt array (e.g. 1e2 cm**(-3))
- Neutral hydrogen density nHI as unyt array (e.g. 1e-5 cm**(-3))
- Ionised hydrogen density nHII as unyt array (e.g. 1e2 cm**(-3))
- Gas temperature temp as unyt array (e.g. 1e4 K)
- HIAtom configuration: nmax=50 levels, Case B flag, recombination and collisional excitation toggles, cache_path

## Outputs

- Level population density of a specified state (n, l), e.g. 3p population 1.64e-17 cm**(-3)
- Line emissivity for a transition (nupper, nlower), e.g. H alpha 3.451e-21 erg/(cm**3*s)
- solver_validation_results.csv recording reconstructed vs reference values

## How to apply

Initialise hylightpy.HIAtom with nmax=50, caseB=True, recom=True, coll=True and a cache_path for cascade matrix elements. Specify gas conditions as unyt arrays: ne=1e2 cm**(-3), nHI=1e-5 cm**(-3), nHII=1e2 cm**(-3), temp=1e4 K. Solve the steady-state population equations via compute_level_pop(nHII, ne, nHI, temp, n, l) to obtain the population of a chosen state (e.g. 3p: n=3, l=1), then compute the H alpha emissivity with get_line_emissivity(nupper=3, nlower=2). Rationale: HyLight builds the rate-equation system including direct recombination cascade terms and collisional excitation from the ground state; benchmarking shows it reproduces Cloudy Balmer, Paschen and Brackett emissivities to within 1 per cent under typical photoionized nebular conditions. Validate by requiring the reconstructed 3p population (1.64e-17 cm**(-3)) and H alpha emissivity (3.451e-21 erg/(cm**3*s)) to agree with HyLight reference-condition outputs within ~1 per cent, then write results to solver_validation_results.csv.

## Related tools

- **HyLight** (Python atomic model whose HIAtom class solves the Case B level-population rate equations and returns line emissivities) — https://github.com/YuankangLiu/HyLight
- **hylightpy** (PyPI package providing HIAtom, compute_level_pop and get_line_emissivity (version 0.0.21 used in the paper)) — https://github.com/YuankangLiu/HyLight
- **unyt** (Specifies gas density and temperature arrays with units (cm**(-3), K) passed to compute_level_pop and get_line_emissivity)
- **numpy** (Array support used in the HyLight example workflow)
- **Cloudy** (Benchmark reference: HyLight reproduces Cloudy Balmer, Paschen and Brackett emissivities to within 1 per cent)

## Examples

```
import hylightpy, unyt; HI = hylightpy.HIAtom(nmax=50, caseB=True, recom=True, coll=True, cache_path='./'); lp = HI.compute_level_pop(nHII=unyt.unyt_array([1e2],'cm**(-3)'), ne=unyt.unyt_array([1e2],'cm**(-3)'), nHI=unyt.unyt_array([1e-5],'cm**(-3)'), temp=unyt.unyt_array([1e4],'K'), n=3, l=1)
```

## Evaluation signals

- Reconstructed 3p (n=3, l=1) population matches the reference 1.64e-17 cm**(-3) within ~1 per cent
- Computed H alpha (n=3 to n=2) emissivity matches the reference 3.451e-21 erg/(cm**3*s) within ~1 per cent
- Results written to solver_validation_results.csv include both reconstructed and reference values
- HIAtom initialised with caseB=True, recom=True, coll=True and nmax=50 as in the reference workflow
- Outputs carry unyt units consistent with inputs (cm**(-3) for populations, erg/(cm**3*s) for emissivity)

## Limitations

- Accuracy is claimed only under typical photoionised nebular conditions; discrepancies of several tens of per cent arise relative to other published calculations
- Cascade matrix elements are cached to cache_path (default current working directory), so a stale or missing cache can affect runs
- hylightpy version used in the paper is 0.0.21; no version is stated for unyt, so reproducibility requires pinning packages

## Evidence

- [intro] HyLight computes the level population of the excited states in atomic hydrogen and derives the typical recombination line emissivity from those populations.: "calculates the level population of the excited states in atomic hydrogen and hence the typical recombination line emissivity"
- [methods] The function compute_level_pop computes the level population density.: "The function ``compute_level_pop`` computes the level population density."
- [methods] Reference outputs at typical nebular conditions.: "Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s)"
- [methods] Reference 3p population value.: "Level population of 3p state is 1.64e-17 cm**(-3)"
- [abstract] Benchmark accuracy against Cloudy.: "HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent"
- [readme] Initialisation pattern from the README.: [evidence span withheld — anchor status: external_doc]
- [readme] unyt specification of gas conditions from the README.: "We utilise `unyt` package to specify the gas density and temperature."
- [readme] 1 per cent accuracy claim under photoionised nebular conditions.: "The results are accurate under typical photoionised nebular conditions."
