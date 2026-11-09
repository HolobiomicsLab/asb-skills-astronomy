---
name: hydrogen-recombination-line-emissivity-calculation
description: Use when you need to compute hydrogen recombination line emissivities
  (Balmer, Paschen, Brackett) or excited-state (n, l) population densities from nebular
  gas conditions (electron/proton/neutral H densities and temperature).
when_to_use_negative:
- The gas is not atomic hydrogen under photoionised nebular conditions — HyLight is
  a hydrogen-only atomic model validated for typical photoionised nebular conditions
- You need metal or helium line emissivities, or Case A hydrogen recombination where
  direct recombinations to the ground state are kept
- You need Cloudy-grade photoionization modelling with full radiative transfer rather
  than hydrogen level-population/recombination-line emissivities
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: HyLight
  role: Python-based atomic model that calculates hydrogen level populations and line
    emissivities from gas density, temperature, and ionization state
  repo: https://github.com/YuankangLiu/HyLight
- name: hylightpy
  role: PyPI distribution of HyLight providing the HIAtom class with get_line_emissivity
    and compute_level_pop methods
  repo: https://pypi.org/project/hylightpy/
- name: unyt
  role: Used to specify the gas density and temperature as unit-aware arrays passed
    to get_line_emissivity and compute_level_pop
- name: Cloudy
  role: Reference photoionization code used to benchmark HyLight Balmer, Paschen,
    and Brackett emissivities
- name: Python
  role: Runtime environment for the hylightpy workflow
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stag429
    title: The <scp>HyLight</scp> model for hydrogen emission lines in simulated nebulae
  source_content_hash: sha256:14e1960929d68131d955461d29a407cc03f2b0ddd07be22229b7be5379ec8c89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuankangLiu/HyLight
  tools:
  - Cloudy
  - HyLight
  - Python
  - hylightpy
  - matplotlib
  - numpy
  - unyt
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# hydrogen-recombination-line-emissivity-calculation

## Summary

Compute hydrogen recombination line emissivities (e.g. the H alpha n=3→n=2 line) and excited-state level populations from gas density, temperature, and ionization state using the HyLight Python atomic model (hylightpy HIAtom) in Case B with radiative recombination and collisional excitation.

## When to use

Use when you need to compute hydrogen recombination line emissivities (Balmer, Paschen, Brackett) or excited-state (n, l) population densities from nebular gas conditions (electron/proton/neutral H densities and temperature).

## When NOT to use

- The gas is not atomic hydrogen under photoionised nebular conditions — HyLight is a hydrogen-only atomic model validated for typical photoionised nebular conditions
- You need metal or helium line emissivities, or Case A hydrogen recombination where direct recombinations to the ground state are kept
- You need Cloudy-grade photoionization modelling with full radiative transfer rather than hydrogen level-population/recombination-line emissivities

## Inputs

- Gas conditions as unyt arrays: electron density ne (cm**(-3), e.g. 1e2), neutral hydrogen density nHI (cm**(-3), e.g. 1e-5), proton density nHII (cm**(-3), e.g. 1e2), temperature temp (K, e.g. 1e4)
- Transition quantum numbers for get_line_emissivity: nupper and nlower (e.g. 3 and 2 for H alpha)
- Level quantum numbers for compute_level_pop: n and l (e.g. 3 and 1 for the 3p state)
- HIAtom configuration parameters: nmax (e.g. 50), caseB (True), recom (True), coll (True), cache_path

## Outputs

- Line emissivity from get_line_emissivity in erg/(cm**3*s) (e.g. 3.451e-21 for H alpha at the reference conditions)
- Level population density from compute_level_pop in cm**(-3) (e.g. 1.64e-17 for the 3p state at the reference conditions)

## How to apply

Install the package via 'pip install hylightpy' and instantiate the atom as hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'), enabling the recombination cascade and collisional excitation from the ground state over 50 n-levels in Case B. Specify gas conditions as unit-aware unyt arrays, e.g. ne=100 cm**(-3), nHI=1e-5 cm**(-3), nHII=100 cm**(-3), temp=1e4 K. Call HI.get_line_emissivity with these arrays and the transition quantum numbers (nupper=3, nlower=2 for H alpha); call HI.compute_level_pop(nHII=..., ne=..., nHI=..., temp=..., n=3, l=1) for the 3p level population density. Rationale: HyLight solves the steady-state level population rate equations and derives emissivities from those populations, which the paper benchmarks against Cloudy to within 1 per cent for Balmer, Paschen, and Brackett series under typical photoionized nebular conditions.

## Related tools

- **HyLight** (Python-based atomic model that calculates hydrogen level populations and line emissivities from gas density, temperature, and ionization state) — https://github.com/YuankangLiu/HyLight
- **hylightpy** (PyPI distribution of HyLight providing the HIAtom class with get_line_emissivity and compute_level_pop methods) — https://pypi.org/project/hylightpy/
- **unyt** (Used to specify the gas density and temperature as unit-aware arrays passed to get_line_emissivity and compute_level_pop)
- **Cloudy** (Reference photoionization code used to benchmark HyLight Balmer, Paschen, and Brackett emissivities)
- **Python** (Runtime environment for the hylightpy workflow)

## Examples

```
HI = hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'); eps = HI.get_line_emissivity(ne=ne, nHI=nHI, nHII=nHII, temp=temp, nupper=3, nlower=2)
```

## Evaluation signals

- H alpha (n=3→n=2) emissivity at ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, T=1e4 K with nmax=50 Case B must equal 3.451e-21 erg/(cm**3*s) to within printing precision
- 3p (n=3, l=1) level population at the same conditions must equal 1.64e-17 cm**(-3) to within printing precision
- Benchmark against Cloudy reference emissivities: maximum relative difference must be within 1 per cent for each of the Balmer (nupper→2), Paschen (nupper→3), and Brackett (nupper→4) series
- All input densities and temperatures must be passed as unyt arrays with correct units (cm**(-3), K), not bare floats
- Cache path ('./' by default) is writable so cascade matrix elements can be stored and reused

## Limitations

- Accuracy is claimed only under typical photoionised nebular conditions; discrepancies of several tens of per cent arise relative to other published calculations
- Results depend on the configured Case (Case A vs Case B) and on whether radiative (recom) and collisional (coll) processes are enabled
- The number of levels (nmax, e.g. 50) limits the recombination cascade resolution; cascade matrix elements are cached to the specified cache_path
- No version number stated for hylightpy or unyt in the methods documentation (hylightpy 0.0.21 appears only in the results notebook)

## Evidence

- [methods] Reference H alpha emissivity: "Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s)"
- [methods] Reference 3p level population: "Level population of 3p state is 1.64e-17 cm**(-3)"
- [abstract] Benchmark accuracy against Cloudy: "Benchmark comparisons show that HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent"
- [readme] Units via unyt: "We utilise `unyt` package to specify the gas density and temperature."
- [methods] HIAtom initialisation: "HI = hylightpy.HIAtom(nmax = 50, verbose=False, caseB=True,
                      recom=True, coll=True, cache_path='./')"
- [intro] Accuracy scope claim: "The results are accurate under typical photoionised nebular conditions."
