---
name: parameterized-simulation-from-config
description: Use when you need a controlled synthetic time series with a known primary
  frequency and prescribed noise (SNR) against which a period- or frequency-finding
  method can be evaluated.
when_to_use_negative:
- When the input is already an observed real light curve with unknown frequencies
  — this skill generates synthetic data with a known truth, it does not analyse real
  observations
- When the evaluation target requires many observation points or multi-band synthetic
  data beyond the fixed single-band configuration described here
edam_operation: http://edamontology.org/operation_0244
edam_topics: []
tools:
- name: FINKER
  role: Provides the SyntheticLightCurveGenerator class (utils_SYNTHETIC_LIGHTCURVES)
    used to draw the 100 noisy magnitude observations at the fixed parameters, and
    downstream kernel-regression frequency search for validation
  repo: github.com/FiorenSt/FINKER
- name: Python
  role: Runtime in which the generator is imported and invoked to write synthetic_light_curve.csv
- name: GitHub
  role: Source for cloning the FINKER repository to obtain the synthetic light curve
    utilities
  repo: github.com/FiorenSt/FINKER
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:32ee1c9baf9b6ce9afeb3a7db314051b74f20e3b9c80acbfac90eae2accc4892
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - GitHub
  - Python
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# parameterized-simulation-from-config

## Summary

Generates a noisy synthetic light curve from a fixed parameter file (n_points=100, time=10, freq_primary=0.6, snr_primary=7, baseline_magnitude=17.0) using FINKER's SyntheticLightCurveGenerator, providing a ground-truth test case for frequency-recovery evaluation.

## When to use

Use when you need a controlled synthetic time series with a known primary frequency and prescribed noise (SNR) against which a period- or frequency-finding method can be evaluated.

## When NOT to use

- When the input is already an observed real light curve with unknown frequencies — this skill generates synthetic data with a known truth, it does not analyse real observations
- When the evaluation target requires many observation points or multi-band synthetic data beyond the fixed single-band configuration described here

## Inputs

- Fixed parameter file defining n_points, time span, freq_primary (true_freq), snr_primary and baseline_magnitude
- FINKER Python package (utils_SYNTHETIC_LIGHTCURVES module)

## Outputs

- synthetic_light_curve.csv with 100 rows of observation times, magnitudes/fluxes and per-point uncertainties
- Ground-truth true frequency (0.6) for subsequent recovery evaluation

## How to apply

Load the fixed parameter file specifying n_points=100, time=10, freq_primary=0.6 (true_freq), snr_primary=7 and baseline_magnitude=17.0. Import the generator with 'from FINKER.utils_SYNTHETIC_LIGHTCURVES import *' (after 'pip install FINKER' or cloning github.com/FiorenSt/FINKER), instantiate SyntheticLightCurveGenerator, and call generate_synthetic_light_curve with the loaded parameters to draw 100 noisy magnitude observations over a 10-unit time span at the known primary frequency. Write the generated observation times, magnitudes/fluxes and per-point uncertainties to synthetic_light_curve.csv. The known true frequency (0.6) and SNR-controlled noise model then serve as ground truth for downstream frequency searches, e.g. FINKER's parallel_nonparametric_kernel_regression over a freq_list such as np.linspace(0.001,5,50000).

## Related tools

- **FINKER** (Provides the SyntheticLightCurveGenerator class (utils_SYNTHETIC_LIGHTCURVES) used to draw the 100 noisy magnitude observations at the fixed parameters, and downstream kernel-regression frequency search for validation) — github.com/FiorenSt/FINKER
- **Python** (Runtime in which the generator is imported and invoked to write synthetic_light_curve.csv)
- **GitHub** (Source for cloning the FINKER repository to obtain the synthetic light curve utilities) — github.com/FiorenSt/FINKER

## Examples

```
from FINKER.utils_SYNTHETIC_LIGHTCURVES import *; generator.generate_synthetic_light_curve(n_points=100, time=10, freq_primary=0.6, snr_primary=7, baseline_magnitude=17.0)
```

## Evaluation signals

- synthetic_light_curve.csv exists and contains exactly 100 rows with time, flux/magnitude and uncertainty columns
- Noise in the magnitudes reflects the SNR=7 noise model at baseline_magnitude=17.0 over a 10-unit time span
- A subsequent FINKER frequency search (Gaussian kernel, local_constant, custom bandwidth, freq_list=np.linspace(0.001,5,50000)) recovers the true primary frequency 0.6 as best_freq
- Parameters are loaded from the fixed parameter file rather than hard-coded, so the run is reproducible

## Limitations

- The synthetic configuration shown is a single-band, single-frequency test (freq_primary=0.6, snr_primary=7, n_points=100); adapting it to other regimes requires changing the fixed parameter file
- The article's discussion section is empty ('No changelog found.'), so no formal limitations, missing artifacts or reproducibility gaps are documented for this data-generation step

## Evidence

- [results] Synthetic light curve parameters: "n_points=100,
    time=10,
    freq_primary=true_freq,
    snr_primary=7,
    baseline_magnitude=17.0,"
- [results] Test frequency to recover: "# Test frequency to recover
true_freq = 0.6"
- [results] Generator import: "from FINKER.utils_SYNTHETIC_LIGHTCURVES import *"
- [readme] Installation path: "To use FINKER, clone the repository or install with:

bash
pip install FINKER
"
