---
name: relativistic-raytracing-pulse-profile-synthesis
description: Use when reproducing or benchmarking neutron star pulse-profile synthesis
  (e.g., Devarshi et al. 2024 scenarios a–d or OS1 test cases) that requires relativistic
  ray-traced surface integration folded through a detector response matrix.
when_to_use_negative:
- The goal is Bayesian parameter inference on already-observed NICER pulse profiles
  (e.g. X-PSI fitting) rather than forward synthesis of profiles from specified spot
  geometries
- No CUDA-capable GPU and CPU-only runtimes are acceptable — use the devarshi24st_cpu
  / os1_cpu reference binaries directly instead of expecting GPU speedups
- Atmosphere emission outside the NSX hydrogen table grid (or near interpolation boundaries
  where the known systematic bias has not been mitigated) is required
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CUDA / nvcc
  role: GPU kernels for ray-traced surface integration, shared-memory interpolation,
    and response-matrix folding
  repo: github.com/zhoutz/gpu_ppm
- name: make
  role: Builds CPU and GPU binaries (make devarshi24st_cpu, make devarshi24st_gpu)
  repo: github.com/zhoutz/gpu_ppm
- name: Clang++ / g++
  role: C++20 host compiler for CPU reference implementations
  repo: github.com/zhoutz/gpu_ppm
- name: Python3 (numpy, scipy, matplotlib)
  role: Generates lensing/ISM-instrument tables (c_lensing.py, c_ism_inst.py) and
    comparison plots (p_devarshi24st.py)
  repo: github.com/zhoutz/gpu_ppm
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202659305
    title: GPU-accelerated X-ray pulse profile modeling
  source_content_hash: sha256:dbdc18c0e7c107ed56a0e7ea5267e9cceea8a57769af39e57a06407c70e23dc3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CUDA / nvcc
  - Clang++
  - Clang++ / g++
  - Python3
  - Python3 (numpy, scipy, matplotlib)
  - g++
  - make
  - matplotlib
  - numpy
  - nvcc
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# relativistic-raytracing-pulse-profile-synthesis

## Summary

Generates phase-resolved X-ray pulse profiles and energy–phase counts for rotating neutron star surface emission via GPU (CUDA) ray tracing that includes relativistic light bending, time-of-flight delays, Doppler/aberration, oblate star geometry, NSX atmosphere beaming, and NICER instrument response folding, with CPU reference binaries for cross-validation to ~1e-3 relative accuracy.

## When to use

Use when reproducing or benchmarking neutron star pulse-profile synthesis (e.g., Devarshi et al. 2024 scenarios a–d or OS1 test cases) that requires relativistic ray-traced surface integration folded through a detector response matrix.

## When NOT to use

- The goal is Bayesian parameter inference on already-observed NICER pulse profiles (e.g. X-PSI fitting) rather than forward synthesis of profiles from specified spot geometries
- No CUDA-capable GPU and CPU-only runtimes are acceptable — use the devarshi24st_cpu / os1_cpu reference binaries directly instead of expecting GPU speedups
- Atmosphere emission outside the NSX hydrogen table grid (or near interpolation boundaries where the known systematic bias has not been mitigated) is required

## Inputs

- Precomputed GR lensing tables (tables/lensing/<setting>/: u.txt, cos_psi.txt, cos_alpha.txt, cos_alpha_of_u_cos_psi.txt, lf_of_u_cos_psi.txt, cdt_over_R_of_u_cos_alpha.txt)
- Instrument/ISM response tables (tables/ism_inst/<setting>/: num.txt, E_obs.txt, rsp.txt N_CH x N_E_obs matrix)
- NSX hydrogen atmosphere intensities (ext/model_data/nsx_H_v200804.out or nsx_H_v171019.out)
- NICER RMF/ARF files (e.g. nicer-rmf6s-teamonly-array50_full_matrix.txt, nicer-consim135p-teamonly-array50_arf.txt)
- Benchmark pulse-profile data (ext/Pulse_profiles_by_diff_codes/, ext/apjlab5968/)
- Spot morphology / geometry specification (rings, crescents, single circular spots, difference of disks; blackbody or atmosphere spectra)

## Outputs

- Per-scenario counts-vs-phase text files under output/ (e.g. output/os1a_gpu.txt ... os1j_gpu.txt and devarshi24st a-d equivalents)
- GPU-vs-CPU comparison plots from p_devarshi24st.py / p_os1.py
- Relative-accuracy statistics against CPU reference and published benchmarks (~1e-3)

## How to apply

First populate external data by extracting ext.tar.xz and, if absent, generate the precomputed tables with `python3 c_lensing.py` (tables/lensing/) and `python3 c_ism_inst.py` (tables/ism_inst/); these provide GR lensing lookup tables (u, cos_psi, cos_alpha, and interpolated lf/cdt files) and the NICER response matrix (num.txt, E_obs.txt, rsp.txt). Build binaries with make devarshi24st_cpu and make devarshi24st_gpu (C++20 host compiler Clang++ or g++ via CXX=g++; nvcc with -arch=sm_89 for an RTX 4080-class GPU — adjust GPUARCH to your compute capability, e.g. sm_80/sm_90). Execute ./build/devarshi24st_gpu and ./build/devarshi24st_cpu, which ray-trace each surface element using HEALPix-like spherical tiling with dynamic active patch selection, bilinear lensing interpolation on device, cubic Lagrange / linear edge interpolation of cached NSX hydrogen atmosphere intensities, per-element time-of-flight and rotational phase shifts, then fold energy fluxes through the response matrix via a GPU matrix multiply to write per-scenario counts-vs-phase text files under output/. Compare GPU vs CPU with python3 p_devarshi24st.py a|b|c|d; acceptance is ~1e-3 relative agreement between GPU, CPU reference, and the Devarshi et al. (2024)/Bogdanov et al. (2019) benchmarks, including extreme hotspot configurations.

## Related tools

- **CUDA / nvcc** (GPU kernels for ray-traced surface integration, shared-memory interpolation, and response-matrix folding) — github.com/zhoutz/gpu_ppm
- **make** (Builds CPU and GPU binaries (make devarshi24st_cpu, make devarshi24st_gpu)) — github.com/zhoutz/gpu_ppm
- **Clang++ / g++** (C++20 host compiler for CPU reference implementations) — github.com/zhoutz/gpu_ppm
- **Python3 (numpy, scipy, matplotlib)** (Generates lensing/ISM-instrument tables (c_lensing.py, c_ism_inst.py) and comparison plots (p_devarshi24st.py)) — github.com/zhoutz/gpu_ppm

## Examples

```
python3 c_lensing.py; python3 c_ism_inst.py; make devarshi24st_cpu devarshi24st_gpu; ./build/devarshi24st_gpu; ./build/devarshi24st_cpu; python3 p_devarshi24st.py a
```

## Evaluation signals

- GPU-generated pulse profiles reproduce CPU reference and Devarshi et al. (2024) benchmarks to ~1e-3 relative accuracy across scenarios a-d, including extreme hotspot configurations
- Required table files exist: tables/lensing/<setting>/{u.txt, cos_psi.txt, cos_alpha.txt, lf_of_u_cos_psi.txt, cdt_over_R_of_u_cos_alpha.txt} and tables/ism_inst/<setting>/{num.txt, E_obs.txt, rsp.txt}
- output/ contains per-scenario counts-vs-phase text files for both GPU and CPU runs (e.g. os1a_gpu.txt ... os1j_gpu.txt)
- GPU-vs-CPU comparison plots (p_devarshi24st.py a|b|c|d, p_os1.py) show overlapping phase-resolved count curves
- rsp.txt matrix has shape N_CH x N_E_obs consistent with num.txt header (N_CH N_E_obs)

## Limitations

- Known systematic error near atmosphere-table interpolation boundaries; a mixed-order interpolator (cubic Lagrange with linear edge interpolation) substantially reduces but does not eliminate this bias
- Makefile defaults -arch=sm_89 (RTX 4080-class); GPUARCH must be manually set for other compute capabilities, and older nvcc may not support -arch=native
- Requires external data (ext.tar.xz with NSX tables, NICER RMF/ARF, benchmark profiles) redistributed under original Creative Commons licenses — original works must be cited
- Extreme hotspot configurations are difficult to resolve at production settings, motivating the 1e-3 (not machine-precision) validation threshold

## Evidence

- [abstract] GPU vs CPU workflows: "Our framework reproduces established benchmarks to a ∼10 −3 relative accuracy, including the case of extreme hotspot configurations"
- [intro] physics completeness: "phase‑resolved pulse profiles and energy–phase counts for rotating neutron star surface emission with relativistic light bending, time delays, Doppler / aberration effects, oblate star shape,"
- [methods] table generation commands: "python3 c_lensing.py   # generates lensing tables in ./tables/lensing"
- [methods] build toolchain: "CUDA toolkit (nvcc) with support for your GPU architecture (Makefile currently sets `-arch=sm_89`"
- [readme] run Devarshi scenarios: "./build/devarshi24st_cpu   # CPU reference 
./build/devarshi24st_gpu   # GPU version"
- [readme] response folding method: "Instrument response folding (tables/ism_inst/*) to produce channel counts vs phase (simulating detector observation) on the GPU via a matrix multiply."
