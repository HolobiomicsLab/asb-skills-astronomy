---
name: case-b-recombination-atomic-level-population-modelling
description: Use when you need hydrogen recombination line emissivities or excited-state
  (n, l) level population densities for a gas with known electron/proton/neutral-H
  densities and temperature, e.g. H ii nebula conditions like ne=100 cm^-3, T=1e4
  K.
when_to_use_negative:
- The task requires modelling of non-hydrogen atomic species — HyLight is a hydrogen-only
  atomic model
- Conditions fall outside typical photoionised nebular conditions, where the paper
  notes discrepancies of several tens of per cent relative to other published calculations
- A full photoionisation code with line transfer (e.g. Cloudy) is required rather
  than a standalone hydrogen recombination-line model
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0128
tools:
- name: HyLight
  role: Python atomic model that calculates hydrogen level populations and line emissivities
    from gas density, temperature, and ionization state
  repo: https://github.com/YuankangLiu/HyLight
- name: hylightpy
  role: PyPI-installable package exposing HIAtom, get_line_emissivity, and compute_level_pop
    used to run the Case B calculation
  repo: https://pypi.org/project/hylightpy/
- name: unyt
  role: Used to specify unit-aware gas density and temperature inputs (unyt.array.unyt_array
    with cm**(-3) and K units)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stag429
    title: The <scp>HyLight</scp> model for hydrogen emission lines in simulated nebulae
  source_content_hash: sha256:a5dd151018ca1464a5081694db7a67151c198334eea552d3b35ddf734d9dc9f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuankangLiu/HyLight
  tools:
  - HyLight
  - Python
  - hylightpy
  - unyt
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# case-b-recombination-atomic-level-population-modelling

## Summary

Computes hydrogen excited-state level populations and recombination line emissivities (e.g. H alpha n=3→2) using the HyLight Python model (hylightpy) under Case B conditions at specified nebular density and temperature.

## When to use

Use when you need hydrogen recombination line emissivities or excited-state (n, l) level population densities for a gas with known electron/proton/neutral-H densities and temperature, e.g. H ii nebula conditions like ne=100 cm^-3, T=1e4 K.

## When NOT to use

- The task requires modelling of non-hydrogen atomic species — HyLight is a hydrogen-only atomic model
- Conditions fall outside typical photoionised nebular conditions, where the paper notes discrepancies of several tens of per cent relative to other published calculations
- A full photoionisation code with line transfer (e.g. Cloudy) is required rather than a standalone hydrogen recombination-line model

## Inputs

- unyt arrays: electron density ne (cm**(-3)), neutral hydrogen density nHI (cm**(-3)), ionized hydrogen density nHII (cm**(-3))
- unyt array: gas temperature temp (K)
- Line transition quantum numbers (nupper, nlower) for emissivity, or (n, l) for level population

## Outputs

- Line emissivity in erg/(cm**3*s) (e.g. H alpha = 3.451e-21 at the reference conditions)
- Level population density in cm**(-3) (e.g. 3p state = 1.64e-17)

## How to apply

Install hylightpy via pip and import it together with unyt for unit-aware quantities. Instantiate the atom model as hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'), enabling radiative recombination and collisional excitation from the ground state with 50 n-levels in Case B. Define conditions as unyt arrays (ne, nHI, nHII in cm**(-3); temp in K, e.g. 100, 1e-5, 100 cm**(-3) and 1e4 K) and call HI.get_line_emissivity(ne=ne, nHI=nHI, nHII=nHII, temp=temp, nupper=3, nlower=2) for the H alpha emissivity, and HI.compute_level_pop(nHII=nHII, ne=ne, nHI=nHI, temp=temp, n=3, l=1) for the 3p population density. Cascade matrix elements are cached in the specified cache_path, so reruns at the same atom configuration are faster. Validate against the documented reference values: emissivity 3.451e-21 erg/(cm**3*s) and 3p population 1.64e-17 cm**(-3).

## Related tools

- **HyLight** (Python atomic model that calculates hydrogen level populations and line emissivities from gas density, temperature, and ionization state) — https://github.com/YuankangLiu/HyLight
- **hylightpy** (PyPI-installable package exposing HIAtom, get_line_emissivity, and compute_level_pop used to run the Case B calculation) — https://pypi.org/project/hylightpy/
- **unyt** (Used to specify unit-aware gas density and temperature inputs (unyt.array.unyt_array with cm**(-3) and K units))

## Examples

```
import hylightpy, unyt; HI = hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'); eps = HI.get_line_emissivity(ne=unyt.array.unyt_array([1e2],'cm**(-3)'), nHI=unyt.array.unyt_array([1e-5],'cm**(-3)'), nHII=unyt.array.unyt_array([1e2],'cm**(-3)'), temp=unyt.array.unyt_array([1e4],'K'), nupper=3, nlower=2)
```

## Evaluation signals

- H alpha (n=3→2) emissivity at ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, T=1e4 K with nmax=50 Case B must equal 3.451e-21 erg/(cm**3*s) to printing precision
- 3p (n=3, l=1) level population at the same conditions must equal 1.64e-17 cm**(-3) to printing precision
- Outputs carry correct units: emissivity in erg/(cm**3*s) and population density in cm**(-3)
- Benchmark check: Balmer, Paschen, and Brackett emissivities should agree with Cloudy to within 1 per cent under typical photoionized nebular conditions
- Cascade matrix cache files are created in the configured cache_path (e.g. './') after first run

## Limitations

- Results accurate under typical photoionised nebular conditions; discrepancies of several tens of per cent arise relative to other published calculations elsewhere
- Hydrogen-only model: cannot treat other atoms/ions
- The hylightpy version used in the paper's examples is 0.0.21; no version pinning for unyt is documented
- Case selection (Case B with caseB=True) and inclusion of recombination/collisional processes must be set at initialisation; different settings change results

## Evidence

- [abstract] HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent: "Benchmark comparisons show that HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent"
- [methods] Reference H alpha emissivity: "Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s)"
- [methods] HIAtom initialisation: "HI = hylightpy.HIAtom(nmax = 50, verbose=False, caseB=True,
                      recom=True, coll=True, cache_path='./')"
- [methods] Level population reference value: "Level population of 3p state is 1.64e-17 cm**(-3)"
- [readme] README usage of unyt for inputs: "We utilise `unyt` package to specify the gas density and temperature."
