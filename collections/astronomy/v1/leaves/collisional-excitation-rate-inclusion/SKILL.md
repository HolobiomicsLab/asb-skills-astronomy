---
name: collisional-excitation-rate-inclusion
description: Use when reconstructing HyLight Case B hydrogen level-population solvers
  or line emissivities for photoionised nebular gas where collisional excitation from
  the ground state must enter the rate-equation system (ne up to ~1e2 cm**(-3), T
  ~1e4 K).
when_to_use_negative:
- When modelling gas conditions far outside typical photoionised nebular conditions
  (e.g. very high densities where the Case B assumption or the 1-per-cent Cloudy agreement
  no longer holds)
- When a Case A solution is required — the reference workflow uses caseB=True; switching
  the case changes the physical model
- When only recombination cascades matter and the question explicitly excludes collisional
  processes; setting coll=False would then be the intended configuration rather than
  this skill
edam_operation: http://edamontology.org/operation_0244
tools:
- name: HyLight
  role: Atomic model providing the HIAtom class whose coll=True option adds collisional
    excitation from the ground state to the level-population rate equations
  repo: https://github.com/YuankangLiu/HyLight
- name: hylightpy
  role: Pip-installable Python package exposing HyLight's HIAtom (nmax=50, verbose=False,
    caseB=True, recom=True, coll=True); version 0.0.21
  repo: https://github.com/YuankangLiu/HyLight
- name: unyt
  role: Specifies gas density and temperature with units (cm**(-3), K) as unyt_array
    inputs to compute_level_pop and get_line_emissivity
- name: numpy
  role: Numerical support for solving and evaluating the level-population system
- name: Cloudy
  role: Benchmark reference; HyLight with radiative and collisional processes enabled
    reproduces Cloudy emissivities to within 1 per cent
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stag429
    title: The <scp>HyLight</scp> model for hydrogen emission lines in simulated nebulae
  source_content_hash: sha256:a7c8246afe07ba676ef68e6ba93eb026e8b730a2276b988c0755caf02a3f8a9b
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
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# collisional-excitation-rate-inclusion

## Summary

Enables collisional-excitation terms (coll=True) when solving hydrogen level populations in HyLight's HIAtom, so that the steady-state rate equations include collisional excitation from the ground state alongside radiative recombination cascades. This is essential for reproducing reference outputs such as the 3p population of 1.64e-17 cm**(-3) and H alpha emissivity of 3.451e-21 erg/(cm**3*s) at ne=100 cm**(-3), T=1e4 K.

## When to use

Use when reconstructing HyLight Case B hydrogen level-population solvers or line emissivities for photoionised nebular gas where collisional excitation from the ground state must enter the rate-equation system (ne up to ~1e2 cm**(-3), T ~1e4 K).

## When NOT to use

- When modelling gas conditions far outside typical photoionised nebular conditions (e.g. very high densities where the Case B assumption or the 1-per-cent Cloudy agreement no longer holds)
- When a Case A solution is required — the reference workflow uses caseB=True; switching the case changes the physical model
- When only recombination cascades matter and the question explicitly excludes collisional processes; setting coll=False would then be the intended configuration rather than this skill

## Inputs

- Gas conditions as unyt arrays: electron density ne = 1e2 cm**(-3), neutral hydrogen density nHI = 1e-5 cm**(-3), proton density nHII = 1e2 cm**(-3), temperature temp = 1e4 K
- HIAtom configuration options: nmax = 50 levels, caseB = True, recom = True, coll = True, cache_path for cascade matrix elements
- Target state quantum numbers (e.g. n=3, l=1) or transition levels (nupper=3, nlower=2)

## Outputs

- Level population density of a specific excited state (e.g. 3p population of 1.64e-17 cm**(-3) at the reference conditions)
- Line emissivity for a transition (e.g. H alpha, n=3 to n=2, of 3.451e-21 erg/(cm**3*s) at the reference conditions)
- Validation results (e.g. solver_validation_results.csv) comparing reconstructed values against HyLight reference-condition outputs

## How to apply

Initialise the HIAtom class with coll=True (together with caseB=True, recom=True, nmax=50) so the rate equations for atomic hydrogen include collisional excitation from the ground state in addition to direct recombination cascade terms. Specify gas conditions with unyt arrays: ne=1e2 cm**(-3), nHI=1e-5 cm**(-3), nHII=1e2 cm**(-3), temp=1e4 K. Solve the steady-state population equations, then evaluate the population of a specific state (e.g. n=3, l=1) via compute_level_pop or derive the H alpha (n=3 to n=2) emissivity via get_line_emissivity. The rationale is that HyLight's accuracy benchmark against Cloudy (agreement to within 1 per cent for Balmer, Paschen, and Brackett emissivities under typical photoionized nebular conditions) is achieved with both radiative and collisional processes enabled; omitting coll=True would bias excited-state populations. Validate by comparing the 3p population to the reference 1.64e-17 cm**(-3) and H alpha emissivity to 3.451e-21 erg/(cm**3*s) within approximately 1 per cent.

## Related tools

- **HyLight** (Atomic model providing the HIAtom class whose coll=True option adds collisional excitation from the ground state to the level-population rate equations) — https://github.com/YuankangLiu/HyLight
- **hylightpy** (Pip-installable Python package exposing HyLight's HIAtom (nmax=50, verbose=False, caseB=True, recom=True, coll=True); version 0.0.21) — https://github.com/YuankangLiu/HyLight
- **unyt** (Specifies gas density and temperature with units (cm**(-3), K) as unyt_array inputs to compute_level_pop and get_line_emissivity)
- **numpy** (Numerical support for solving and evaluating the level-population system)
- **Cloudy** (Benchmark reference; HyLight with radiative and collisional processes enabled reproduces Cloudy emissivities to within 1 per cent)

## Examples

```
import hylightpy, unyt; HI = hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'); levelpop = HI.compute_level_pop(nHII=unyt.array.unyt_array([1e2],'cm**(-3)'), ne=unyt.array.unyt_array([1e2],'cm**(-3)'), nHI=unyt.array.unyt_array([1e-5],'cm**(-3)'), temp=unyt.array.unyt_array([1e4],'K'), n=3, l=1)
```

## Evaluation signals

- Reconstructed 3p (n=3, l=1) level population agrees with the HyLight reference value 1.64e-17 cm**(-3) within approximately 1 per cent
- Reconstructed H alpha (n=3 to n=2) emissivity agrees with the reference 3.451e-21 erg/(cm**3*s) at ne=100 cm**(-3), nHI=1e-5 cm**(-3), nHII=100 cm**(-3), T=1e4 K within approximately 1 per cent
- HIAtom is initialised with coll=True (and caseB=True, recom=True, nmax=50); enabling flags without collisional terms should change excited-state populations and would signal misconfiguration
- Inputs are passed as unyt arrays with correct units (cm**(-3) for densities, K for temperature), not bare floats
- Validation results written to solver_validation_results.csv contain both reconstructed values and percentage differences against reference outputs

## Limitations

- HyLight's quoted accuracy (agreement with Cloudy to within 1 per cent) holds under typical photoionized nebular conditions; discrepancies of several tens of per cent arise relative to other published calculations
- The collisional terms are for excitation from the ground state in a Case B hydrogen atom with up to nmax=50 levels; physics outside this configuration is not covered by the reference workflow
- Cascade matrix elements are cached to a cache_path (default current working directory); a wrong or stale cache path could affect results

## Evidence

- [methods] HIAtom initialisation with collisional flag: "HI = hylightpy.HIAtom(nmax = 50, verbose=False, caseB=True,
                      recom=True, coll=True, cache_path='./')"
- [abstract] HyLight benchmark accuracy: "Benchmark comparisons show that HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent"
- [methods] Reference H alpha emissivity: "Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s)"
- [methods] Reference 3p level population: "Level population of 3p state is 1.64e-17 cm**(-3)"
- [readme] unyt usage for gas conditions: "We utilise `unyt` package to specify the gas density and temperature."
- [readme] README initialisation showing coll option: "HI = hylightpy.HIAtom(nmax = 50, verbose=True, caseB=True, 
                      recom=True, coll=True, 
                      cache_path='./')"
