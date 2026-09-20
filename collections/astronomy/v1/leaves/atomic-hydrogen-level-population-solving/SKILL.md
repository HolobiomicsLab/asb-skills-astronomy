---
name: atomic-hydrogen-level-population-solving
description: Use when you need level population densities of excited hydrogen states
  (e.g. the 3p state) or hydrogen recombination line emissivities (e.g. H alpha) for
  gas with specified ne, nHI, nHII, and temperature.
when_to_use_negative:
- If the gas conditions are far outside typical photoionised nebular conditions (e.g.
  densities/temperatures where Case B recombination assumptions fail), since HyLight's
  accuracy claim is stated for typical photoionised nebular conditions only.
- If you need emission from other species or non-hydrogen recombination lines — the
  HIAtom model covers only atomic hydrogen excited-state populations.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3450
tools:
- name: HyLight
  role: Atomic hydrogen model that computes level populations and line emissivities;
    accessed via the hylightpy Python package
  repo: https://github.com/YuankangLiu/HyLight
- name: hylightpy
  role: PyPI package providing the HIAtom class with compute_level_pop and get_line_emissivity
    methods (version 0.0.21 used in the article)
  repo: https://github.com/YuankangLiu/HyLight
- name: unyt
  role: Used to specify gas density and temperature with physical units (cm**(-3),
    K) as input arrays
- name: numpy
  role: Imported for numerical handling in the example workflow
- name: Cloudy
  role: Benchmark code used to validate HyLight emissivities to within 1 per cent
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stag429
    title: The <scp>HyLight</scp> model for hydrogen emission lines in simulated nebulae
  source_content_hash: sha256:83ab2dad3877c84831badc18a286cd7425258a00bec06dc58451cf26917ed39d
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
  - http://edamontology.org/topic_3450
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# atomic-hydrogen-level-population-solving

## Summary

Solves the steady-state rate equations for excited-state level populations of atomic hydrogen using HyLight (hylightpy), given gas density, temperature, and ionization state, and derives recombination line emissivities such as H alpha. Used to reproduce or benchmark Case B hydrogen recombination emission in photoionised nebulae.

## When to use

Use when you need level population densities of excited hydrogen states (e.g. the 3p state) or hydrogen recombination line emissivities (e.g. H alpha) for gas with specified ne, nHI, nHII, and temperature.

## When NOT to use

- If the gas conditions are far outside typical photoionised nebular conditions (e.g. densities/temperatures where Case B recombination assumptions fail), since HyLight's accuracy claim is stated for typical photoionised nebular conditions only.
- If you need emission from other species or non-hydrogen recombination lines — the HIAtom model covers only atomic hydrogen excited-state populations.

## Inputs

- Electron density ne as a unyt array (e.g. [1e2] cm**(-3))
- Neutral hydrogen density nHI as a unyt array (e.g. [1e-5] cm**(-3))
- Ionised hydrogen density nHII as a unyt array (e.g. [1e2] cm**(-3))
- Gas temperature temp as a unyt array (e.g. [1e4] K)
- HIAtom configuration: nmax=50 levels, caseB=True, recom=True, coll=True, cache_path

## Outputs

- Level population density of a specified (n, l) state, e.g. 3p = 1.64e-17 cm**(-3)
- Line emissivity for a transition, e.g. H alpha (n=3 to n=2) = 3.451e-21 erg/(cm**3*s)

## How to apply

Install hylightpy via pip and import hylightpy; initialise the atom as HI = hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'), where nmax=50 sets the number of hydrogen levels, caseB selects Case B recombination, and recom/coll enable radiative recombination cascades and collisional excitation from the ground state. Specify gas conditions with unyt arrays: ne = unyt.array.unyt_array([1e2], 'cm**(-3)'), nHI = unyt.array.unyt_array([1e-5], 'cm**(-3)'), nHII = unyt.array.unyt_array([1e2], 'cm**(-3)'), temp = unyt.array.unyt_array([1e4], 'K'). Call HI.compute_level_pop(nHII=nHII, ne=ne, nHI=nHI, temp=temp, n=3, l=1) to solve the steady-state population equations for a specific state (here 3p). Compute line emissivities with HI.get_line_emissivity(ne=ne, nHI=nHI, nHII=nHII, temp=temp, nupper=3, nlower=2) for H alpha. Validate against the HyLight reference values: 3p population 1.64e-17 cm**(-3) and H alpha emissivity 3.451e-21 erg/(cm**3*s), expecting agreement within about 1 per cent.

## Related tools

- **HyLight** (Atomic hydrogen model that computes level populations and line emissivities; accessed via the hylightpy Python package) — https://github.com/YuankangLiu/HyLight
- **hylightpy** (PyPI package providing the HIAtom class with compute_level_pop and get_line_emissivity methods (version 0.0.21 used in the article)) — https://github.com/YuankangLiu/HyLight
- **unyt** (Used to specify gas density and temperature with physical units (cm**(-3), K) as input arrays)
- **numpy** (Imported for numerical handling in the example workflow)
- **Cloudy** (Benchmark code used to validate HyLight emissivities to within 1 per cent)

## Examples

```
import hylightpy, unyt; HI = hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'); ne=unyt.array.unyt_array([1e2],'cm**(-3)'); nHI=unyt.array.unyt_array([1e-5],'cm**(-3)'); nHII=unyt.array.unyt_array([1e2],'cm**(-3)'); temp=unyt.array.unyt_array([1e4],'K'); levelpop = HI.compute_level_pop(nHII=nHII, ne=ne, nHI=nHI, temp=temp, n=3, l=1); eps = HI.get_line_emissivity(ne=ne, nHI=nHI, nHII=nHII, temp=temp, nupper=3, nlower=2)
```

## Evaluation signals

- Reconstructed 3p level population matches the HyLight reference 1.64e-17 cm**(-3) within ~1 per cent
- Computed H alpha emissivity matches the reference 3.451e-21 erg/(cm**3*s) within ~1 per cent
- Benchmark against Cloudy: Balmer, Paschen, and Brackett emissivities agree within 1 per cent under typical photoionized nebular conditions
- Unit consistency verified: inputs specified as unyt arrays with units cm**(-3) and K
- Results written to solver_validation_results.csv and both values pass the ~1 per cent acceptance criterion

## Limitations

- Accuracy is claimed only under typical photoionised nebular conditions; other regimes may show discrepancies of several tens of per cent relative to other published calculations
- Discrepancies of several tens of per cent arise relative to other published calculations
- No version number stated for hylightpy or unyt in the methods section (article results used hylightpy 0.0.21)

## Evidence

- [methods] Initialise HIAtom with nmax levels, Case B, recombination and collisional excitation: "HI = hylightpy.HIAtom(nmax = 50, verbose=False, caseB=True,
                      recom=True, coll=True, cache_path='./')"
- [methods] compute_level_pop computes the level population density: "The function ``compute_level_pop`` computes the level population density."
- [methods] Reference H alpha emissivity under specified nebular conditions: "Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s)"
- [methods] Reference 3p level population: "Level population of 3p state is 1.64e-17 cm**(-3)"
- [abstract] Benchmark accuracy against Cloudy: "Benchmark comparisons show that HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent"
- [readme] unyt used for unitful gas conditions: "We utilise `unyt` package to specify the gas density and temperature."
- [readme] hylightpy installation command: "The package can be installed via
``npip install hylightpy
"
