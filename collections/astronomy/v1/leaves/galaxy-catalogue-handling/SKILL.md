---
name: galaxy-catalogue-handling
description: Use when you need to load a galaxy or simulated halo catalogue (e.g.
  cat.dat) into CosmoBolognaLib Catalogue objects, build a random catalogue, apply
  mass/Mmin cuts, or count pairs/triplets for 2PCF/3PCF.
when_to_use_negative:
- Only a subset of the CBL functions has a Python wrapper — if the required catalogue-handling
  or clustering method is not wrapped, work directly in C++ rather than forcing the
  Python interface.
- If you need an archival/pinned software version or a DOI-referenced release, note
  the CBL download is a .tar archive from a personal institutional webpage with no
  changelog — use a different tool if strict provenance is required.
- If the analysis is not catalogue-based clustering in configuration space (e.g. you
  need CMB-level Boltzmann solving like MGCAMB), this catalogue-handling skill is
  the wrong scope.
tools:
- name: CosmoBolognaLib
  role: Core C++ library providing Catalogue/Random Catalogue classes, random-catalogue
    construction, and parallel chain-mesh pair/triplet counting for 2PCF and 3PCF
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: GSL
  role: External numerical library backing CBL functions; Numerical Recipes was fully
    substituted by GSL from CBL V3.0
- name: C++
  role: Primary language in which catalogue-handling and TwoPointCorrelation/ThreePointCorrelation
    classes are used
- name: Python
  role: Optional SWIG-wrapped interface (compiled with 'make python') for calling
    a subset of CBL catalogue-handling functions
  repo: https://github.com/federicomarulli/CosmoBolognaLib
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:c9d421494168fd18e20d2711177c6d2c148188f4ee8f699d32437ff4f6ab1d9a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - CosmoBolognaLib
  - GSL
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# galaxy-catalogue-handling

## Summary

Handling of real and simulated galaxy/halo catalogues with the CosmoBolognaLib (CBL) C++/Python libraries: building Catalogue and Random Catalogue objects, applying sub-catalogue cuts, and feeding them into two- and three-point statistics measured with a parallel chain-mesh algorithm in configuration space.

## When to use

Use when you need to load a galaxy or simulated halo catalogue (e.g. cat.dat) into CosmoBolognaLib Catalogue objects, build a random catalogue, apply mass/Mmin cuts, or count pairs/triplets for 2PCF/3PCF.

## When NOT to use

- Only a subset of the CBL functions has a Python wrapper — if the required catalogue-handling or clustering method is not wrapped, work directly in C++ rather than forcing the Python interface.
- If you need an archival/pinned software version or a DOI-referenced release, note the CBL download is a .tar archive from a personal institutional webpage with no changelog — use a different tool if strict provenance is required.
- If the analysis is not catalogue-based clustering in configuration space (e.g. you need CMB-level Boltzmann solving like MGCAMB), this catalogue-handling skill is the wrong scope.

## Inputs

- galaxy catalogue file (e.g. cat.dat) read into a cbl::Catalogue
- simulated halo catalogue (e.g. HOD catalogue with volume 3.677700e+07 [Mpc/h]^3 for 1/8 of sphere)
- random catalogue (constructed via CBL random-catalogue algorithms or pre-existing)
- parameter file for sub-catalogue cuts (e.g. minimum mass threshold 'Mmin')
- local directory structure: random/, pairs/, output/

## Outputs

- Catalogue and Random Catalogue objects (vector of Galaxy objects)
- filtered sub-catalogue after mass cut-off
- DD/DR/RR pair-count files written under the pairs/ directory
- measured ξ(r) table (e.g. output/xi.dat) for bins in the 1–50 h⁻¹Mpc range
- 3PCF table with ζ and Q values as a function of θ

## How to apply

Instantiate a vector of Galaxy objects from the catalogue file and wrap it in a cbl::Catalogue, then construct the associated random catalogue using CBL's built-in random-catalogue algorithms. If a tracer selection is needed, apply a sub-catalogue cut on a variable such as mass (e.g. trcat = temp.sub_catalogue(cbl.Var__Mass_, Mmin, max_mass, False)). For clustering statistics, configure an angle-averaged TwoPointCorrelation1D object with linearly spaced separation bins (e.g. r = 1–50 h⁻¹Mpc) and count DD, DR and RR pairs — or DDD, DDR, DRR and RRR triplets at fixed triangle sides with varying angle θ — using the parallel chain-mesh pair/triplet-counting algorithm, writing pair files under pairs/. Combine the counts with the Landy-Szalay estimator (2PCF) or the Szapudi-Szalay estimator (3PCF, with reduced Q = ζ/(ξ12ξ23 + ξ23ξ31 + ξ31ξ12)) and write results to output/. CBL is written in C++ and backed by GSL numerics; only a subset currently has a Python wrapper compiled with 'make python'.

## Related tools

- **CosmoBolognaLib** (Core C++ library providing Catalogue/Random Catalogue classes, random-catalogue construction, and parallel chain-mesh pair/triplet counting for 2PCF and 3PCF) — https://github.com/federicomarulli/CosmoBolognaLib
- **GSL** (External numerical library backing CBL functions; Numerical Recipes was fully substituted by GSL from CBL V3.0)
- **C++** (Primary language in which catalogue-handling and TwoPointCorrelation/ThreePointCorrelation classes are used)
- **Python** (Optional SWIG-wrapped interface (compiled with 'make python') for calling a subset of CBL catalogue-handling functions) — https://github.com/federicomarulli/CosmoBolognaLib

## Examples

```
trcat = temp.sub_catalogue(cbl.Var__Mass_, param.findDouble('Mmin'), temp.Max(cbl.Var__Mass_), False)
```

## Evaluation signals

- output/xi.dat exists and contains ξ(r) values for every bin in the 1–50 h⁻¹Mpc range, with the curve decreasing smoothly at large separations as expected for a clustered galaxy sample.
- Pair-count files are written under the pairs/ directory for DD, DR and RR counts before estimator combination.
- The 3PCF output table contains ζ and Q columns in the estimator-derived format with finite values and Q of order unity across the θ bins.
- The sub-catalogue cut is applied with the minimum mass from the parameter file: sub_catalogue(cbl.Var__Mass_, Mmin, max_mass, False).
- Catalogue properties print consistently, e.g. 'Catalogue Volume: 3.677700e+07 [Mpc/h]^3' for the HOD 1/8-sphere catalogue.

## Limitations

- Only a subset of the CBL currently has a Python wrapper; full coverage is deferred to a future version.
- No fixed version/release identifier is given for the download — only a CosmoBolognaLib.tar archive at a personal institutional URL (no DOI/zenodo), and the repository contains no changelog.
- The 2PCF example depends on local input files and directories (cat.dat, random/, pairs/, output/) that are not described or hosted alongside the paper.
- External dependencies (GSL, FFTW, OpenMP, formerly Numerical Recipes 3rd ed.) must be installed before compiling with 'make'; on MAC OS use 'make SYS=`MAC`'.
- When constructing a Cosmology object, default cosmological parameters are used unless set explicitly (e.g. Omega_M=0.319, sigma8=0.83, h=0.67).

## Evidence

- [intro] the CBL provide highly optimised algorithms to measure two-point (2PCF) and three-point correlation functions (3PCF), exploiting a specifically designed parallel chain-mesh algorithm to count pairs: [evidence span withheld — anchor status: not_found]
