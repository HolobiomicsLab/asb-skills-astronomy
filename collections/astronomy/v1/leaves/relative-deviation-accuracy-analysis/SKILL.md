---
name: relative-deviation-accuracy-analysis
description: Use when validating a physics/chemistry emission model against published
  reference predictions by comparing per-line emissivities and summarising the maximum
  relative deviation per spectral series.
when_to_use_negative:
- The gas is not under typical photoionized nebular conditions (e.g. temperatures
  or densities far from T=1e4 K, ne=100 cm^-3), since the 1 per cent agreement criterion
  was established only for those conditions
- The goal is comparison against other published hydrogen-atom calculations rather
  than Cloudy; the paper notes discrepancies of several tens of per cent can arise
  in that case
- Line transitions outside the Balmer, Paschen, or Brackett series covered by the
  Cloudy reference set are needed
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: HyLight
  role: Hydrogen atomic model computing level populations and line emissivities being
    benchmarked against Cloudy
  repo: https://github.com/YuankangLiu/HyLight
- name: hylightpy
  role: PyPI package used to instantiate HIAtom and compute line emissivities via
    get_line_emissivity
  repo: https://github.com/YuankangLiu/HyLight
- name: Cloudy
  role: Reference photoionization code whose emissivity predictions define the benchmark
    target values
- name: unyt
  role: Specifies gas densities and temperature with units for the emissivity calculation
- name: numpy
  role: Computes per-line relative percentage differences and per-series maximum deviations
- name: matplotlib
  role: Plotting of benchmark comparison results
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stag429
    title: The <scp>HyLight</scp> model for hydrogen emission lines in simulated nebulae
  source_content_hash: sha256:682a797d6a9891976b94c481d895f3d77085eb9ce878d7aed93913ea24d17766
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# relative-deviation-accuracy-analysis

## Summary

Quantifies the accuracy of a computational model (HyLight) against a reference code (Cloudy) by computing per-line relative percentage differences in hydrogen recombination line emissivities and checking agreement against a stated 1 per cent threshold for the Balmer, Paschen, and Brackett series.

## When to use

Use when validating a physics/chemistry emission model against published reference predictions by comparing per-line emissivities and summarising the maximum relative deviation per spectral series.

## When NOT to use

- The gas is not under typical photoionized nebular conditions (e.g. temperatures or densities far from T=1e4 K, ne=100 cm^-3), since the 1 per cent agreement criterion was established only for those conditions
- The goal is comparison against other published hydrogen-atom calculations rather than Cloudy; the paper notes discrepancies of several tens of per cent can arise in that case
- Line transitions outside the Balmer, Paschen, or Brackett series covered by the Cloudy reference set are needed

## Inputs

- Cloudy reference emissivity data/scripts for Balmer, Paschen, and Brackett series (from the YuankangLiu/HyLight benchmark folder)
- Nebular conditions specified as unyt arrays: ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, temp=1e4 K
- HyLight emissivities from HI.get_line_emissivity for nupper=2, 3, 4 transitions

## Outputs

- Per-line relative percentage differences between HyLight and Cloudy emissivities
- Maximum relative deviation per series (Balmer, Paschen, Brackett)
- benchmark_comparison_results.csv containing HyLight emissivities, Cloudy reference values, and relative differences

## How to apply

Install HyLight via 'pip install hylightpy' and load the Cloudy reference emissivity data/scripts from the YuankangLiu/HyLight benchmark folder. Initialise the atom as hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'), enabling radiative recombination and collisional excitation in Case B. Define typical photoionized nebular conditions with unyt: ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, T=1e4 K. Compute emissivities with HI.get_line_emissivity for Balmer (nupper=2), Paschen (nupper=3), and Brackett (nupper=4) series transitions covered by the Cloudy reference set. Use numpy to compute the per-line relative percentage difference between HyLight and Cloudy values, summarise the maximum deviation per series, and write results (HyLight emissivities, Cloudy references, relative differences) to benchmark_comparison_results.csv. The benchmark passes only if the maximum relative difference is within 1 per cent for each of the three series, matching the paper's reported agreement criterion.

## Related tools

- **HyLight** (Hydrogen atomic model computing level populations and line emissivities being benchmarked against Cloudy) — https://github.com/YuankangLiu/HyLight
- **hylightpy** (PyPI package used to instantiate HIAtom and compute line emissivities via get_line_emissivity) — https://github.com/YuankangLiu/HyLight
- **Cloudy** (Reference photoionization code whose emissivity predictions define the benchmark target values)
- **unyt** (Specifies gas densities and temperature with units for the emissivity calculation)
- **numpy** (Computes per-line relative percentage differences and per-series maximum deviations)
- **matplotlib** (Plotting of benchmark comparison results)

## Examples

```
import hylightpy, unyt, numpy as np; HI = hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'); eps = HI.get_line_emissivity(ne=unyt.array.unyt_array([1e2],'cm**(-3)'), nHI=unyt.array.unyt_array([1e-5],'cm**(-3)'), nHII=unyt.array.unyt_array([1e2],'cm**(-3)'), temp=unyt.array.unyt_array([1e4],'K'), nupper=3, nlower=2)
```

## Evaluation signals

- Maximum relative difference between HyLight and Cloudy emissivities is within 1 per cent for each of the Balmer, Paschen, and Brackett series
- benchmark_comparison_results.csv contains three columns/values: HyLight emissivities, Cloudy reference values, and relative differences per line
- H alpha (3->2) emissivity at ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, T=1e4 K is close to 3.451e-21 erg/(cm**3*s)
- Computed 3p level population is close to 1.64e-17 cm**(-3) at the same conditions
- All emissivities carry physical units (erg/(cm**3*s)) via the unyt-wrapped inputs

## Limitations

- The 1 per cent agreement with Cloudy holds under typical photoionized nebular conditions; accuracy under extreme thermal conditions is not established by this benchmark
- Discrepancies of several tens of per cent can arise relative to other published calculations, so Cloudy is not a universal ground truth
- The benchmark covers only the Balmer, Paschen, and Brackett transitions included in the Cloudy reference set

## Evidence

- [abstract] HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent under typical photoionized nebular conditions: "Benchmark comparisons show that HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent under typical photoionized nebular conditions"
- [methods] Emissivity validation value: "Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s)"
- [methods] Level population validation value: "Level population of 3p state is 1.64e-17 cm**(-3)"
- [methods] HIAtom initialisation parameters: "HI = hylightpy.HIAtom(nmax = 50, verbose=False, caseB=True,
                      recom=True, coll=True, cache_path='./')"
- [methods] get_line_emissivity usage: "eps = HI.get_line_emissivity(ne=ne,
                             nHI=nHI, 
                             nHII=nHII, 
                             temp=temp, 
                             nupper=3,"
- [readme] unyt for physical conditions: "We utilise `unyt` package to specify the gas density and temperature."
