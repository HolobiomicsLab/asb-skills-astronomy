---
name: balmer-paschen-brackett-series-transitions
description: Use when calculating or validating hydrogen recombination line emissivities
  (Hα, Paschen, Brackett) for a photoionized nebula with given electron/proton densities
  and temperature, or when benchmarking an atomic model against Cloudy.
when_to_use_negative:
- The target transitions or physical conditions fall outside the Cloudy reference
  set (e.g. very different densities/temperatures), so the 1 per cent agreement criterion
  cannot be evaluated
- A full photoionization/radiative-transfer calculation is needed rather than an atomic-model
  emissivity lookup — use Cloudy itself instead
- Line emission from species other than atomic hydrogen is required; HyLight models
  only hydrogen level populations
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: HyLight
  role: Atomic model computing hydrogen level populations and recombination line emissivities
    for the Balmer, Paschen, and Brackett series
  repo: https://github.com/YuankangLiu/HyLight
- name: hylightpy
  role: PyPI package providing HIAtom and get_line_emissivity used to compute the
    series emissivities (version 0.0.21 cited in the results section)
  repo: https://github.com/YuankangLiu/HyLight
- name: Cloudy
  role: Reference photoionization code whose Balmer, Paschen, and Brackett emissivity
    predictions serve as the benchmark target (agreement within 1 per cent)
- name: unyt
  role: Specifies gas density and temperature as physical unyt arrays passed to get_line_emissivity
- name: numpy
  role: Computes per-line relative percentage differences and maximum deviation per
    series against the Cloudy reference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stag429
    title: The <scp>HyLight</scp> model for hydrogen emission lines in simulated nebulae
  source_content_hash: sha256:32aa9fb725ad48bfbef6fb6cdaf9344496a3a51d475d66486d0edbc81011d70c
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# balmer-paschen-brackett-series-transitions

## Summary

Compute hydrogen recombination line emissivities for the Balmer (nupper→2), Paschen (nupper→3), and Brackett (nupper→4) series using the HyLight atomic model (hylightpy.HIAtom) and benchmark them against Cloudy reference predictions, requiring agreement within 1 per cent per series.

## When to use

Use when calculating or validating hydrogen recombination line emissivities (Hα, Paschen, Brackett) for a photoionized nebula with given electron/proton densities and temperature, or when benchmarking an atomic model against Cloudy.

## When NOT to use

- The target transitions or physical conditions fall outside the Cloudy reference set (e.g. very different densities/temperatures), so the 1 per cent agreement criterion cannot be evaluated
- A full photoionization/radiative-transfer calculation is needed rather than an atomic-model emissivity lookup — use Cloudy itself instead
- Line emission from species other than atomic hydrogen is required; HyLight models only hydrogen level populations

## Inputs

- Cloudy reference emissivity data/scripts from the YuankangLiu/HyLight repository benchmark folder
- Nebular conditions specified with unyt: ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, temp=1e4 K
- Series definitions: Balmer nupper→2, Paschen nupper→3, Brackett nupper→4 (Case B)

## Outputs

- HyLight line emissivities for Balmer, Paschen, and Brackett transitions (e.g. Hα = 3.451e-21 erg/(cm**3*s) at the stated conditions)
- Per-line relative percentage differences versus Cloudy, summarised as maximum deviation per series
- benchmark_comparison_results.csv containing HyLight emissivities, Cloudy reference values, and relative differences

## How to apply

Install the HyLight package via 'pip install hylightpy' and initialise the hydrogen atom with hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'), enabling Case B radiative recombination and collisional excitation from the ground state. Specify the typical photoionized nebular conditions with the unyt package: ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, temp=1e4 K. Call HI.get_line_emissivity for the Balmer (nupper=2), Paschen (nupper=3), and Brackett (nupper=4) series transitions covered by the Cloudy reference set in the YuankangLiu/HyLight benchmark folder. Compute per-line relative percentage differences against the Cloudy reference values with numpy and summarise the maximum deviation per series; the benchmark succeeds only if the maximum relative difference is within 1 per cent for each series. Optionally load the Cloudy reference emissivity data/scripts shipped in the repository benchmark folder rather than regenerating them.

## Related tools

- **HyLight** (Atomic model computing hydrogen level populations and recombination line emissivities for the Balmer, Paschen, and Brackett series) — https://github.com/YuankangLiu/HyLight
- **hylightpy** (PyPI package providing HIAtom and get_line_emissivity used to compute the series emissivities (version 0.0.21 cited in the results section)) — https://github.com/YuankangLiu/HyLight
- **Cloudy** (Reference photoionization code whose Balmer, Paschen, and Brackett emissivity predictions serve as the benchmark target (agreement within 1 per cent))
- **unyt** (Specifies gas density and temperature as physical unyt arrays passed to get_line_emissivity)
- **numpy** (Computes per-line relative percentage differences and maximum deviation per series against the Cloudy reference)

## Examples

```
HI = hylightpy.HIAtom(nmax=50, verbose=False, caseB=True, recom=True, coll=True, cache_path='./'); eps = HI.get_line_emissivity(ne=ne, nHI=nHI, nHII=nHII, temp=temp, nupper=3, nlower=2)
```

## Evaluation signals

- Maximum relative difference between HyLight and Cloudy emissivities is within 1 per cent for each of the Balmer, Paschen, and Brackett series
- Sanity check at the canonical conditions: Hα (n=3→2) emissivity equals 3.451e-21 erg/(cm**3*s) at ne=100 cm^-3, nHI=1e-5 cm^-3, nHII=100 cm^-3, T=1e4 K
- Level population check: 3p state population density is 1.64e-17 cm**(-3) at the same conditions via compute_level_pop
- benchmark_comparison_results.csv is written with HyLight emissivities, Cloudy reference values, and relative differences for all lines in the reference set

## Limitations

- Discrepancies of several tens of per cent arise relative to other published calculations, so the 1 per cent agreement is specific to the Cloudy comparison
- Results are validated under typical photoionised nebular conditions; accuracy under extreme density/temperature regimes is not established by this benchmark
- No version number stated for unyt in the methods section (hylightpy 0.0.21 is cited in the results section), so reproducibility of dependency versions is incomplete

## Evidence

- [abstract] Benchmark comparisons show that HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent under typical photoionized nebular conditions: "Benchmark comparisons show that HyLight reproduces Cloudy predictions for Balmer, Paschen, and Brackett emissivities to within 1 per cent under typical photoionized nebular conditions"
- [methods] HIAtom initialisation snippet: "HI = hylightpy.HIAtom(nmax = 50, verbose=False, caseB=True,
                      recom=True, coll=True, cache_path='./')"
- [methods] get_line_emissivity call: "eps = HI.get_line_emissivity(ne=ne,
                             nHI=nHI, 
                             nHII=nHII, 
                             temp=temp, 
                             nupper=3,"
- [methods] Hα emissivity value: "Emissivity in the n=3 to n=2 line is 3.451e-21 erg/(cm**3*s)"
- [methods] unyt usage: "We utilise ``unyt`` package to specify the gas density and temperature."
- [results] hylightpy version: "hylightpy.__version__"
- [readme] README installation: "The package can be installed via
``npip install hylightpy
"
- [readme] README nebular conditions: [evidence span withheld — anchor status: external_doc]
