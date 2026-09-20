---
name: statistical-equilibrium-solver-invocation
description: Use when you need hydrogen recombination line emissivities or excited-state
  (n, l) level populations for a gas parcel with known electron/proton/neutral-H densities
  and temperature, e.g. H alpha at ne=100 cm^-3, T=1e4 K.
when_to_use_negative:
- The gas is not atomic hydrogen recombination/line emission (e.g. metal-line or molecular
  cooling is needed) — HyLight models hydrogen only.
- Case A (optically thin to Lyman photons) or fully non-equilibrium conditions outside
  typical photoionised nebular conditions are required and cannot be represented by
  the caseB/recom/coll flags.
- A full photoionization code treatment (e.g. Cloudy) of the ionization balance is
  needed; HyLight takes ne, nHI, nHII, and T as given inputs.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3452
tools:
- name: HyLight
  role: Atomic model whose HIAtom class solves hydrogen level populations and returns
    line emissivities and level populations
  repo: https://github.com/YuankangLiu/HyLight
- name: hylightpy
  role: pip-installable Python package exposing HIAtom, get_line_emissivity, and compute_level_pop
  repo: https://pypi.org/project/hylightpy/
- name: unyt
  role: Unit-aware arrays used to specify gas density and temperature inputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stag429
    title: The <scp>HyLight</scp> model for hydrogen emission lines in simulated nebulae
  source_content_hash: sha256:10fd5fd0a78a7e8cc9d585578b12791f39b0f2b1101b71527edd1fb617923940
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
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# statistical-equilibrium-solver-invocation

## Summary

Invoke the HyLight (hylightpy) HIAtom statistical equilibrium model to compute hydrogen excited-state level populations and recombination line emissivities (e.g. H alpha, n=3 to n=2) from gas density, temperature, and ionization state under nebular conditions.

## When to use

Use when you need hydrogen recombination line emissivities or excited-state (n, l) level populations for a gas parcel with known electron/proton/neutral-H densities and temperature, e.g. H alpha at ne=100 cm^-3, T=1e4 K.

## When NOT to use

- The gas is not atomic hydrogen recombination/line emission (e.g. metal-line or molecular cooling is needed) — HyLight models hydrogen only.
- Case A (optically thin to Lyman photons) or fully non-equilibrium conditions outside typical photoionised nebular conditions are required and cannot be represented by the caseB/recom/coll flags.
- A full photoionization code treatment (e.g. Cloudy) of the ionization balance is needed; HyLight takes ne, nHI, nHII, and T as given inputs.

## Inputs

- unyt arrays of electron density ne (cm**(-3)), neutral hydrogen density nHI (cm**(-3)), ionized hydrogen density nHII (cm**(-3)), and temperature temp (K)
- HIAtom configuration: nmax (number of n-levels, e.g. 50), caseB, recom, coll flags, cache_path
- Quantum numbers of the target transition (nupper, nlower) or level (n, l)

## Outputs

- Line emissivity for the requested transition (e.g. H alpha n=3->n=2) in erg/(cm**3*s)
- Level population density for the requested (n, l) state (e.g. 3p) in cm**(-3)

## How to apply

Install hylightpy via pip and instantiate the hydrogen atom as hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'), enabling radiative recombination and collisional excitation from the ground state with 50 n-levels in Case B. Express gas conditions as unit-aware unyt arrays (e.g. ne=100 cm**(-3), nHI=1e-5 cm**(-3), nHII=100 cm**(-3), temp=1e4 K). Call HI.get_line_emissivity with nupper=3, nlower=2 to obtain the H alpha emissivity, and HI.compute_level_pop with n=3, l=1 to obtain the 3p level population density. The Case B choice (caseB=True) is the standard optically-thick-to-Lyman assumption for photoionized nebulae; hyLight results reproduce Cloudy Balmer, Paschen, and Brackett emissivities to within 1 per cent under typical photoionized nebular conditions.

## Related tools

- **HyLight** (Atomic model whose HIAtom class solves hydrogen level populations and returns line emissivities and level populations) — https://github.com/YuankangLiu/HyLight
- **hylightpy** (pip-installable Python package exposing HIAtom, get_line_emissivity, and compute_level_pop) — https://pypi.org/project/hylightpy/
- **unyt** (Unit-aware arrays used to specify gas density and temperature inputs)

## Examples

```
import hylightpy, unyt; HI = hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'); eps = HI.get_line_emissivity(ne=unyt.array.unyt_array([1e2],'cm**(-3)'), nHI=unyt.array.unyt_array([1e-5],'cm**(-3)'), nHII=unyt.array.unyt_array([1e2],'cm**(-3)'), temp=unyt.array.unyt_array([1e4],'K'), nupper=3, nlower=2)
```

## Evaluation signals

- H alpha emissivity at ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, T=1e4 K with nmax=50, caseB=True must equal 3.451e-21 erg/(cm**3*s).
- The 3p level population under the same conditions must equal 1.64e-17 cm**(-3).
- HIAtom is initialised with the documented flags (nmax=50, caseB=True, recom=True, coll=True) and inputs are unyt arrays with correct units (cm**(-3), K).
- Cross-check against Cloudy: HyLight Balmer, Paschen, and Brackett emissivities agree to within 1 per cent for typical photoionized nebular conditions.

## Limitations

- Discrepancies of several tens of per cent arise relative to other published calculations, even though Cloudy agreement is within 1 per cent.
- Accuracy is claimed only under typical photoionised nebular conditions; unyt version and hylightpy version are not pinned in the methods workflow (hylightpy 0.0.21 appears in results notebooks).
- Cascade matrix elements are cached in cache_path (default current working directory); a stale or missing cache could affect runs.

## Evidence

- [abstract] HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent: "Benchmark comparisons show that HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent"
- [methods] Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s): "Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s)"
- [methods] Level population of 3p state is 1.64e-17 cm**(-3): "Level population of 3p state is 1.64e-17 cm**(-3)"
- [readme] HIAtom initialisation snippet: "HI = hylightpy.HIAtom(nmax = 50, verbose=True, caseB=True,
                      recom=True, coll=True,
                      cache_path='./')"
- [readme] unyt usage for gas conditions: "We utilise `unyt` package to specify the gas density and temperature."
