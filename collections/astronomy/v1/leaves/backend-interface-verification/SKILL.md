---
name: backend-interface-verification
description: Use when a GAMBIT scan (e.g. a DarkBit module-mode run like `gambit -f
  yaml_files/MSSM7.yaml`) is about to be launched and you must confirm the backends
  and scanners it depends on are installed and configured.
when_to_use_negative:
- You are running a DarkBit calculation in stand-alone mode rather than as a GAMBIT
  module, since the GAMBIT backend query machinery does not govern that code path.
- You have not yet retrieved, unpacked and built GAMBIT — verification is meaningless
  until cmake configuration and `make -jn backends` have completed.
- The analysis targets modules unrelated to the backends you plan to check (e.g. ColliderBit-only
  runs do not need DarkSUSY/micrOMEGAs/nulike verification).
edam_operation: null
tools:
- name: GAMBIT
  role: Provides the backend system and the `gambit backends` / `gambit scanners`
    verification commands
  repo: github.com/GambitBSM/gambit
- name: DarkSUSY
  role: Dark-matter backend whose installed status is confirmed during verification
- name: micrOMEGAs
  role: Dark-matter backend whose installed status is confirmed during verification
- name: DDCalc
  role: Direct detection likelihood package checked as a DarkBit backend
- name: gamLike
  role: Gamma-ray likelihood calculator checked as a DarkBit backend
- name: nulike
  role: Neutrino telescope likelihood package checked as a DarkBit backend
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:7fa09d6ca65710cc63f288da9dea6483600e624c363c04de60a9225213f6c656
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DDCalc
  - DarkBit
  - DarkSUSY
  - GAMBIT
  - gamLike
  - micrOMEGAs
  - nulike
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# backend-interface-verification

## Summary

Verifies that the external codes GAMBIT interfaces to through its 'backend' system (e.g. DarkSUSY, micrOMEGAs, DDCalc, gamLike, nulike for DarkBit) and the statistical scanners are correctly downloaded, configured and linked before launching a scan.

## When to use

Use when a GAMBIT scan (e.g. a DarkBit module-mode run like `gambit -f yaml_files/MSSM7.yaml`) is about to be launched and you must confirm the backends and scanners it depends on are installed and configured.

## When NOT to use

- You are running a DarkBit calculation in stand-alone mode rather than as a GAMBIT module, since the GAMBIT backend query machinery does not govern that code path.
- You have not yet retrieved, unpacked and built GAMBIT — verification is meaningless until cmake configuration and `make -jn backends` have completed.
- The analysis targets modules unrelated to the backends you plan to check (e.g. ColliderBit-only runs do not need DarkSUSY/micrOMEGAs/nulike verification).

## Inputs

- Built GAMBIT installation (built via cmake, with scanners and backends compiled)
- gambit backends / gambit scanners CLI queries
- Target module YAML input file (e.g. yaml_files/MSSM7.yaml) indicating which backends are needed

## Outputs

- Status listing of installed backends (e.g. DarkSUSY, micrOMEGAs, DDCalc, gamLike, nulike) and scanners
- Go/no-go decision on whether the planned module-mode scan can be launched

## How to apply

After building GAMBIT with cmake (`cd build; cmake ..; make -jn gambit`) and compiling its scanners and backends (`make -jn scanners`, `make -jn backends`), run `gambit backends` and `gambit scanners` to inspect which external codes — for DarkBit these include DarkSUSY, micrOMEGAs, DDCalc, gamLike and nulike — are present and usable. Check the reported status/version of each backend required by the target module's YAML input file; if a dependency is missing, use the backend download/configuration scripts GAMBIT ships (or install optional helpers like 'axel' to speed up downloads, and MPI for parallel sampling). Only proceed to the module-mode run once the required backends and scanners are confirmed, because the dark matter observables and likelihoods are computed by these external codes via the backend system rather than by GAMBIT core itself.

## Related tools

- **GAMBIT** (Provides the backend system and the `gambit backends` / `gambit scanners` verification commands) — github.com/GambitBSM/gambit
- **DarkSUSY** (Dark-matter backend whose installed status is confirmed during verification)
- **micrOMEGAs** (Dark-matter backend whose installed status is confirmed during verification)
- **DDCalc** (Direct detection likelihood package checked as a DarkBit backend)
- **gamLike** (Gamma-ray likelihood calculator checked as a DarkBit backend)
- **nulike** (Neutrino telescope likelihood package checked as a DarkBit backend)

## Examples

```
gambit backends && gambit scanners
```

## Evaluation signals

- `gambit backends` completes and reports the backends required by the module's YAML file as installed/configured.
- `gambit scanners` completes and lists the statistical scanner to be used for the scan.
- The subsequent module-mode run (`gambit -f yaml_files/MSSM7.yaml`) produces no backend-not-found or scanner-not-found fatal errors.
- GAMBIT's backend download/configuration scripts report success for any backend that had to be fetched before verification.
- Compulsory dependencies listed in the README (CMake, Boost, GSL, Eigen, LAPACK, Python with yaml module, etc.) are satisfied so backends can actually be built.

## Limitations

- cmake may fail to find some dependencies on some systems without guidance; the README points to cluster configuration examples at gambit.hepforge.org.
- Several useful helpers are optional rather than compulsory: MPI is required for parallel sampling, HDF5/h5py for the hdf5 printer, and axel only speeds up backend downloads — scans may be limited if these are absent.
- Backend availability depends on successful external code downloads and configuration; a failed download silently leaves the backend unusable until re-run.
- Python module requirements differ per backend (e.g. Cython for classy, scipy/numpy for MontePython or DarkAges), so one verification pass may not cover every optional backend.

## Evidence

- [methods] Verify installed backends and scanners: "gambit backends
  gambit scanners"
- [intro] Backend system description: "a flexible system for interfacing external codes (the "backend" system)"
- [abstract] DarkBit backend interfaces: "along with interfaces to DarkSUSY , micrOMEGAs , and the neutrino telescope likelihood package nulike"
- [readme] README on backend scripts: "GAMBIT contains interfaces to various external codes, along with scripts for downloading and configuring them."
- [readme] README build prerequisites: "GAMBIT is built using the CMake system. The following libraries and packages must be installed prior to configuration:"
