---
name: landy-szalay-estimator-application
description: Use when the task requires measuring the angle-averaged two-point correlation
  function from a galaxy catalogue and an associated random catalogue in configuration
  space, e.g. over linearly spaced separation bins spanning r = 1–50 h⁻¹Mpc.
when_to_use_negative:
- The analysis targets three-point statistics or other clustering summaries rather
  than the two-point correlation function — use the corresponding 3PCF classes instead.
- Only a Python environment is available and the needed function is outside the subset
  of CBL with a Python wrapper, since only a subset of the CBL currently has a Python
  wrapper.
- The catalogue needs sub-selection first (e.g. a minimum-mass cut on tracers); apply
  the filter step before measuring ξ(r).
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: CosmoBolognaLib
  role: Provides the TwoPointCorrelation classes, chain-mesh pair counters, random-catalogue
    construction, and the Landy–Szalay estimator used to measure ξ(r).
  repo: github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Language in which the CBL libraries are fully written and in which the 2PCF
    example program is compiled and run.
  repo: github.com/federicomarulli/CosmoBolognaLib
- name: GSL
  role: External dependency; from V3.0 the CBL classes for two-point statistics rely
    on GSL functions (Numerical Recipes fully substituted).
  repo: github.com/federicomarulli/CosmoBolognaLib
- name: Python
  role: Optional high-level access to CBL two-point statistics through SWIG-generated
    wrappers (compiled with 'make python').
  repo: github.com/federicomarulli/CosmoBolognaLib
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:668fbd49fa7ea1b65c1028616c3a971dfee70d6c6ae86d564f4ed6565c86cf8f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C++
  - Cosmo Bologna Lib
  - CosmoBolognaLib
  - GSL
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# landy-szalay-estimator-application

## Summary

Measures the angle-averaged two-point correlation function ξ(r) from galaxy and random catalogues by combining normalised data–data (DD), random–random (RR) and data–random (DR) pair counts with the Landy–Szalay estimator, implemented via CosmoBolognaLib's parallel chain-mesh pair-counting classes.

## When to use

Use when the task requires measuring the angle-averaged two-point correlation function from a galaxy catalogue and an associated random catalogue in configuration space, e.g. over linearly spaced separation bins spanning r = 1–50 h⁻¹Mpc.

## When NOT to use

- The analysis targets three-point statistics or other clustering summaries rather than the two-point correlation function — use the corresponding 3PCF classes instead.
- Only a Python environment is available and the needed function is outside the subset of CBL with a Python wrapper, since only a subset of the CBL currently has a Python wrapper.
- The catalogue needs sub-selection first (e.g. a minimum-mass cut on tracers); apply the filter step before measuring ξ(r).

## Inputs

- Galaxy catalogue file (cat.dat) read into a CBL Catalogue object
- Associated random catalogue constructed with CBL random-catalogue algorithms
- Separation-bin configuration (linearly spaced bins over r = 1–50 h⁻¹Mpc)

## Outputs

- DD, DR and RR pair-count files under the pairs/ directory
- Angle-averaged two-point correlation function ξ(r) table written to output/xi.dat

## How to apply

Load the compiled CosmoBolognaLib (C++ libraries relying on GSL functions, optionally wrapped in Python via SWIG) and include the TwoPointCorrelation classes in the example program. Read the galaxy catalogue (cat.dat) into a Catalogue object and construct the associated random catalogue using CBL's random-catalogue algorithms. Configure an angle-averaged TwoPointCorrelation1D object with linearly spaced separation bins spanning r = 1–50 h⁻¹Mpc, then count DD, DR and RR pairs using CBL's parallel chain-mesh pair-counting algorithm, writing pair files under the pairs/ directory. Combine the pair counts with the Landy–Szalay estimator (Eq. 4) to obtain ξ(r) at each bin centre, and write the measured ξ(r) table to output/xi.dat. The rationale for the chain-mesh approach is that CBL provides highly optimised algorithms exploiting a specifically designed parallel chain-mesh algorithm to count pairs for 2PCF measurement.

## Related tools

- **CosmoBolognaLib** (Provides the TwoPointCorrelation classes, chain-mesh pair counters, random-catalogue construction, and the Landy–Szalay estimator used to measure ξ(r).) — github.com/federicomarulli/CosmoBolognaLib
- **C++** (Language in which the CBL libraries are fully written and in which the 2PCF example program is compiled and run.) — github.com/federicomarulli/CosmoBolognaLib
- **GSL** (External dependency; from V3.0 the CBL classes for two-point statistics rely on GSL functions (Numerical Recipes fully substituted).) — github.com/federicomarulli/CosmoBolognaLib
- **Python** (Optional high-level access to CBL two-point statistics through SWIG-generated wrappers (compiled with 'make python').) — github.com/federicomarulli/CosmoBolognaLib

## Examples

```
cd CosmoBolognaLib/ && make && make python
```

## Evaluation signals

- output/xi.dat exists and contains ξ(r) values for every bin in the 1–50 h⁻¹Mpc range.
- The measured ξ(r) curve decreases smoothly at large separations, as expected for a clustered galaxy sample.
- Pair-count files for DD, DR and RR are present under the pairs/ directory for each separation bin.
- The estimator uses all three normalised pair counts (DD, RR, DR) combined per Landy–Szalay (Eq. 4), not a simple DD/RR ratio.
- The CBL build completes successfully, including GSL-dependent two-point statistics classes (compile with 'make' in the CosmoBolognaLib/ folder).

## Limitations

- Only a subset of the CBL has a Python wrapper; full coverage is deferred to a future version, so some 2PCF functionality may require C++.
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) that are not described or hosted with the paper.
- No fixed version/release identifier is given for the CBL download in the paper; the download location was a personal institutional URL rather than an archival repository.
- On MAC OS the build command differs: use 'make SYS=`MAC`'.

## Evidence

- [intro] the CBL provide highly optimised algorithms to measure two-point (2PCF) and three-point correlation functions (3PCF), exploiting a specifically designed parallel chain-mesh algorithm to count pairs: [evidence span withheld — anchor status: not_found]
