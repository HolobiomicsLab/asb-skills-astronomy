---
name: cloudy-reference-benchmark-comparison
description: 'Use when validating a hydrogen emission-line model against Cloudy: compute
  Balmer (nupper=2), Paschen (nupper=3), and Brackett (nupper=4) emissivities and
  check each series stays within 1 per cent of reference values.'
when_to_use_negative:
- Input conditions lie outside typical photoionized nebular regimes (e.g. very high
  densities or temperatures) where the paper's 1 per cent agreement claim was not
  established
- The goal is comparison against other published calculations, which the paper notes
  can differ from HyLight by several tens of per cent
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: HyLight
  role: Hydrogen atomic model whose emissivities are benchmarked against Cloudy
  repo: https://github.com/YuankangLiu/HyLight
- name: hylightpy
  role: PyPI package used to instantiate HIAtom and compute line emissivities
  repo: https://github.com/YuankangLiu/HyLight
- name: Cloudy
  role: Reference code providing benchmark emissivities for the comparison
- name: unyt
  role: Specifies gas density and temperature as physical units
- name: numpy
  role: Computes per-line relative percentage differences
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stag429
    title: The <scp>HyLight</scp> model for hydrogen emission lines in simulated nebulae
  source_content_hash: sha256:64a3b6ee3365601529b82cb8220a578f2575f4710bc5d51cdfc3212c9b94dafc
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cloudy-reference-benchmark-comparison

## Summary

Benchmarks HyLight's hydrogen recombination line emissivities against Cloudy reference predictions for the Balmer, Paschen, and Brackett series under typical photoionized nebular conditions, validating agreement to within 1 per cent per series.

## When to use

Use when validating a hydrogen emission-line model against Cloudy: compute Balmer (nupper=2), Paschen (nupper=3), and Brackett (nupper=4) emissivities and check each series stays within 1 per cent of reference values.

## When NOT to use

- Input conditions lie outside typical photoionized nebular regimes (e.g. very high densities or temperatures) where the paper's 1 per cent agreement claim was not established
- The goal is comparison against other published calculations, which the paper notes can differ from HyLight by several tens of per cent

## Inputs

- Cloudy reference emissivity data/scripts from the YuankangLiu/HyLight repository benchmark folder
- Nebular conditions as unyt arrays: ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, temp=1e4 K
- Series transition definitions (Balmer nupper=2, Paschen nupper=3, Brackett nupper=4)

## Outputs

- HyLight emissivities per line (erg/(cm**3*s))
- Per-line relative percentage differences versus Cloudy
- Maximum relative deviation per series
- benchmark_comparison_results.csv

## How to apply

Install HyLight via 'pip install hylightpy' and initialise the hydrogen atom with hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'), enabling radiative recombination and collisional excitation from the ground state in Case B. Specify typical photoionized nebular conditions with unyt: ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, T=1e4 K. Use HI.get_line_emissivity for each Balmer, Paschen, and Brackett transition covered by the Cloudy reference set (loaded from the YuankangLiu/HyLight benchmark folder). Compute per-line relative percentage differences with numpy and summarise the maximum deviation per series; the benchmark passes if every series has a maximum relative difference within 1 per cent, matching the paper's criterion. Write HyLight values, Cloudy references, and differences to benchmark_comparison_results.csv.

## Related tools

- **HyLight** (Hydrogen atomic model whose emissivities are benchmarked against Cloudy) — https://github.com/YuankangLiu/HyLight
- **hylightpy** (PyPI package used to instantiate HIAtom and compute line emissivities) — https://github.com/YuankangLiu/HyLight
- **Cloudy** (Reference code providing benchmark emissivities for the comparison)
- **unyt** (Specifies gas density and temperature as physical units)
- **numpy** (Computes per-line relative percentage differences)

## Examples

```
HI = hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'); eps = HI.get_line_emissivity(ne=ne, nHI=nHI, nHII=nHII, temp=temp, nupper=3, nlower=2)
```

## Evaluation signals

- Maximum relative difference between HyLight and Cloudy emissivities is within 1 per cent for each of the Balmer, Paschen, and Brackett series
- H alpha emissivity at ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, T=1e4 K equals 3.451e-21 erg/(cm**3*s)
- 3p level population at the same conditions equals 1.64e-17 cm**(-3)
- benchmark_comparison_results.csv contains HyLight values, Cloudy reference values, and relative differences for every line in the reference set
- hylightpy.__version__ reports the version used for the benchmark (0.0.21 in the paper)

## Limitations

- Agreement was established only under typical photoionized nebular conditions; no claim of 1 per cent accuracy outside this regime
- Discrepancies of several tens of per cent arise between HyLight and other published calculations, so Cloudy-only agreement does not guarantee universal accuracy
- No version numbers were stated for unyt; hylightpy version 0.0.21 was used in the paper's results

## Evidence

- [abstract] Benchmark agreement criterion: "Benchmark comparisons show that HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent under typical photoionized nebular conditions"
- [methods] HIAtom initialisation: "HI = hylightpy.HIAtom(nmax = 50, verbose=False, caseB=True,
                      recom=True, coll=True, cache_path='./')"
- [methods] get_line_emissivity call: "eps = HI.get_line_emissivity(ne=ne,
                             nHI=nHI, 
                             nHII=nHII, 
                             temp=temp, 
                             nupper=3,"
- [methods] Reference H alpha emissivity: "Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s)"
- [readme] README nebular conditions: "ne=unyt.array.unyt_array([1e2], 'cm**(-3)')
nHI=unyt.array.unyt_array([1e-5], 'cm**(-3)')
nHII=unyt.array.unyt_array([1e2], 'cm**(-3)')
temp=unyt.array.unyt_array([1e4], 'K')"
- [abstract] Discrepancy caveat: "while discrepancies of several tens of per cent arise relative to other published calculations"
