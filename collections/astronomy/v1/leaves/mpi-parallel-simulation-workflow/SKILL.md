---
name: mpi-parallel-simulation-workflow
description: Use when the task requires installing and running an MPI-parallel cosmological
  N-body code (CONCEPT/GADGET-2) from deposited parameter files and validating it
  via power-spectrum cross-comparison up to k ~ 13 h/Mpc.
when_to_use_negative:
- If the analysis targets only linear-theory spectra, run CLASS directly — no N-body
  simulation run is needed and the workflow's cost is unjustified
- For very high-resolution runs (k_Nyquist well above ~5 Mpc/h), CONCEPT is expected
  to be slower than GADGET-2, so a GADGET-4/GADGET-2-only workflow may be preferable
  on runtime grounds
- If core counts far above what the FFTW slab decomposition supports are required
  for the long-range force computation, the CONCEPT path will not scale as needed
edam_operation: http://edamontology.org/operation_0244
tools:
- name: CONCEPT 1.0
  role: Primary Python-based MPI-parallel N-body code whose simulations are installed,
    tested, and executed; comparison target for reproducing Figure 5
  repo: https://github.com/jmd-dk/concept
- name: GADGET-2
  role: Cross-code reference used to form the P_concept/P_gadget-2 ratio; also target
    of the bundled GADGET test suite
- name: class
  role: Linear Einstein-Boltzmann reference for initial conditions / linear spectra,
    installed and patched by the install script
  repo: https://github.com/lesgourg/class_public
- name: FFTW
  role: MPI-parallelised, real, 3D, in-place transforms used by CONCEPT; its slab
    decomposition constrains core counts
- name: OpenMPI
  role: MPI runtime the code is linked against and run with (OpenMPI 4.0.3 in the
    paper)
- name: Docker
  role: Alternative deployment route via Docker Hub images of CONCEPT 1.0 instead
    of source installation
  repo: https://hub.docker.com/r/jmddk/concept
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:636d9eea949701e007abe7b11272fecd05c48c93c031a57a420973d1fdf18686
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jmd-dk/concept
  tools:
  - CONCEPT 1.0
  - Docker
  - FFTW
  - GADGET-2
  - NumPy
  - OpenMPI
  - Python
  - class
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8148148148148148
n_claims: 27
---

# mpi-parallel-simulation-workflow

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

End-to-end workflow for setting up, building, MPI-parallel test-verification, and running the CONCEPT 1.0 N-body code (with GADGET-2 as cross-check) to reproduce cross-code matter power-spectrum agreement benchmarks across 2048/1024/512/256 Mpc/h boxes at N=1024^3 particles.

## When to use

Use when the task requires installing and running an MPI-parallel cosmological N-body code (CONCEPT/GADGET-2) from deposited parameter files and validating it via power-spectrum cross-comparison up to k ~ 13 h/Mpc.

## When NOT to use

- If the analysis targets only linear-theory spectra, run CLASS directly — no N-body simulation run is needed and the workflow's cost is unjustified
- For very high-resolution runs (k_Nyquist well above ~5 Mpc/h), CONCEPT is expected to be slower than GADGET-2, so a GADGET-4/GADGET-2-only workflow may be preferable on runtime grounds
- If core counts far above what the FFTW slab decomposition supports are required for the long-range force computation, the CONCEPT path will not scale as needed

## Inputs

- Simulation parameter files from the concept1.0-data repository (CONCEPT and GADGET-2 runs, 2048/1024/512/256 Mpc/h boxes, N=1024^3)
- CONCEPT install script defining the ordered dependency chain and patch functions (patch_class, redefine_class, set_optimizations)
- High-precision and x_r = 5.5 x_s parameter variants
- Identical k-binning definition spanning large scales up to k ~ 13 h/Mpc

## Outputs

- MPI-parallel particle snapshots (GADGET-format output) from CONCEPT and GADGET-2 runs
- Matter power spectra P(k) per box and per code on matched k-bins
- Cross-code ratio spectra P_concept/P_gadget-2 and per-regime maximum-deviation metrics (e.g. written to power_spectrum_ratios.csv)

## How to apply

Install CONCEPT via the published install script, which downloads and builds the full dependency chain in order: zlib, GSL, Perl, MPICH/OpenMPI, HDF5, FFTW, ncurses, OpenSSL, libffi, OpenBLAS, Python, CLASS (patched via patch_class/redefine_class), FFTW 2, and GADGET. After building, verify the installation by running the bundled test suites (basic test, GADGET test, full CO*N*CEPT test suite), logging output e.g. with tee. Retrieve simulation parameter files for the N=1024^3 CONCEPT and GADGET-2 runs in the 2048/1024/512/256 Mpc/h boxes from the github.com/Aarhus Cosmology/concept1.0-data repository, then run matched MPI-parallel simulations (including the high-precision and x_r = 5.5 x_s variants). Compute power spectra from both codes on identical k-bins, form the cross-code ratio P_concept/P_gadget-2 per box, and extract maximum deviation per scale regime: agreement should be well below the per mille level at large/intermediate scales and of order 1% at the smallest scales probed (k ~ 13 h/Mpc, the Nyquist scale of the 256 Mpc/h box). Note CONCEPT scales well up to at least 1024 CPU cores, but the FFTW slab decomposition limits the number of cores in the long-range force computation, and CONCEPT is several times faster than GADGET-2 only for k_Nyquist < 5 Mpc/h.

## Related tools

- **CONCEPT 1.0** (Primary Python-based MPI-parallel N-body code whose simulations are installed, tested, and executed; comparison target for reproducing Figure 5) — https://github.com/jmd-dk/concept
- **GADGET-2** (Cross-code reference used to form the P_concept/P_gadget-2 ratio; also target of the bundled GADGET test suite)
- **class** (Linear Einstein-Boltzmann reference for initial conditions / linear spectra, installed and patched by the install script) — https://github.com/lesgourg/class_public
- **FFTW** (MPI-parallelised, real, 3D, in-place transforms used by CONCEPT; its slab decomposition constrains core counts)
- **OpenMPI** (MPI runtime the code is linked against and run with (OpenMPI 4.0.3 in the paper))
- **Docker** (Alternative deployment route via Docker Hub images of CONCEPT 1.0 instead of source installation) — https://hub.docker.com/r/jmddk/concept

## Examples

```
"${concept}" -t "basic" 2>&1 | tee "${concept_test_log}"
```

## Evaluation signals

- All three test suites pass after install: basic test, GADGET test, and the full CO*N*CEPT test suite (e.g. via '${concept}' -t 'basic')
- Regenerated P_concept/P_gadget-2 ratios agree well below the per mille level at large and intermediate scales in each box
- At the smallest probed scales (k ~ 13 h/Mpc, Nyquist scale of the 256 Mpc/h box), the cross-code deviation is of order 1%, matching Tables 1–3 and Figure 5
- Ratio spectra are computed on identical k-bins for both codes so the comparison is bin-by-bin, and metrics are emitted to power_spectrum_ratios.csv
- Runs complete under MPI with the deposited parameter files, including the high-precision and x_r = 5.5 x_s variants

## Limitations

- CONCEPT uses the standard FFTW slab-decomposition, which limits the number of CPU cores participating in the long-range force computation
- CONCEPT is several times faster than GADGET-2 only for low to medium resolutions (k_Nyquist < 5 Mpc/h); the opposite holds for higher-resolution runs
- Performance degrades for strongly clustered systems due to load imbalance
- P^3M gravity is noted as inefficient and GADGET snapshot output format is limited
- No DOI or archival record is provided for code releases; only GitHub tags (e.g. v1.0.1) and commit-comparison links
- A buggy time-stepping limiter for fluid components existed prior to the v0.2.0 fix, so older versions should be avoided

## Evidence

- [abstract] Data repository: [evidence span withheld — anchor status: not_found]
- [abstract] Scaling and MPI runtime: [evidence span withheld — anchor status: not_found]
- [readme] Docker alternative: [evidence span withheld — anchor status: not_found]
