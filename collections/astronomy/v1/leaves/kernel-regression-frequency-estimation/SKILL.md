---
name: kernel-regression-frequency-estimation
description: Use when a time series (e.g. a synthetic or observed light curve with
  magnitudes, times, and uncertainties) must have its primary periodic frequency recovered,
  especially with few observations (down to ~10) or across varying SNR levels.
when_to_use_negative:
- The data are not a time series with a periodic signal — kernel regression frequency
  search assumes folding at candidate frequencies is meaningful.
- Observation timestamps or per-point measurement uncertainties are unavailable, since
  the weighting system accounts for measurement uncertainties.
- The candidate frequency grid does not span the true frequency (default search covers
  0.001–5).
edam_operation: http://edamontology.org/operation_0337
tools:
- name: FINKER
  role: 'Core library: runs the parallel nonparametric kernel regression frequency
    search (kernel folding, local_constant/local_linear regression, bandwidth selection)'
  repo: github.com/FiorenSt/FINKER
- name: Python
  role: Driver environment for building the frequency grid (np.linspace), generating
    synthetic light curves, and collecting best_freq/freq_err results
  repo: github.com/FiorenSt/FINKER
- name: GitHub
  role: Distribution and contribution channel; install via pip or clone the repository
  repo: github.com/FiorenSt/FINKER
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:e95c260566672fa51bc6cc6134fb42a1e2065c36c731acb303ff6a848d4f7ba1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - GitHub
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# kernel-regression-frequency-estimation

## Summary

Applies FINKER's nonparametric kernel regression on folded time-series data at candidate frequencies, selecting the optimal frequency by minimising squared residuals — designed for accurate frequency recovery in noisy, sparse astronomical light curves.

## When to use

Use when a time series (e.g. a synthetic or observed light curve with magnitudes, times, and uncertainties) must have its primary periodic frequency recovered, especially with few observations (down to ~10) or across varying SNR levels.

## When NOT to use

- The data are not a time series with a periodic signal — kernel regression frequency search assumes folding at candidate frequencies is meaningful.
- Observation timestamps or per-point measurement uncertainties are unavailable, since the weighting system accounts for measurement uncertainties.
- The candidate frequency grid does not span the true frequency (default search covers 0.001–5).

## Inputs

- numpy arrays: t_observed (observation times), y_observed (magnitudes/fluxes), uncertainties (measurement errors)
- numpy array of candidate frequencies to search (e.g. np.linspace(0.001, 5, 50000))
- synthetic light-curve parameters such as n_points=100, time=10, freq_primary=0.6, snr_primary, baseline_magnitude=17.0

## Outputs

- best_freq: the estimated optimal frequency
- freq_err: the frequency error estimate
- result_dict: detailed per-frequency regression results
- optionally a compiled results table (e.g. snr_sweep_results.csv with SNR level, estimated frequency, frequency error)

## How to apply

Generate or load the light curve, then instantiate FINKER and call parallel_nonparametric_kernel_regression with t_observed, y_observed, and uncertainties as numpy arrays. Sweep a frequency list such as freq_list = np.linspace(0.001, 5, 50000), using kernel_type='gaussian', regression_type='local_constant', and bandwidth_method='custom' for computational efficiency (Silverman's, Scott's, or Adaptive bandwidth are alternatives). The method folds the data at each candidate frequency, applies nonparametric kernel regression, and selects the best frequency by minimising squared residuals; the inherent weighting system accounts for measurement uncertainties. Record the returned best_freq and freq_err and compare against the known true frequency (e.g. 0.6) to validate recovery.

## Related tools

- **FINKER** (Core library: runs the parallel nonparametric kernel regression frequency search (kernel folding, local_constant/local_linear regression, bandwidth selection)) — github.com/FiorenSt/FINKER
- **Python** (Driver environment for building the frequency grid (np.linspace), generating synthetic light curves, and collecting best_freq/freq_err results) — github.com/FiorenSt/FINKER
- **GitHub** (Distribution and contribution channel; install via pip or clone the repository) — github.com/FiorenSt/FINKER

## Examples

```
from FINKER.utils_FINKER import *; finker = FINKER(); best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(t_observed=t, y_observed=y, uncertainties=dy, freq_list=np.linspace(0.001,5,50000), kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', n_jobs=-2)
```

## Evaluation signals

- Estimated best_freq remains close to the known true frequency (e.g. 0.6) within the reported freq_err.
- Output table (e.g. snr_sweep_results.csv) contains one row per configured SNR level with columns SNR level, estimated frequency, and frequency error.
- Performance is superior in accuracy and robustness compared to a baseline such as the conditional entropy period-finding algorithm.
- Recovery succeeds with sparse data — the method requires fewer observations to reliably identify significant frequencies (as few as ~10).

## Limitations

- Resilience against noise and adaptation to varying dataset complexity are claimed in the abstract, but the discussion section is empty ('No changelog found.' placeholder), so no documented limitations or reproducibility gaps are available from the paper itself.
- Only Gaussian kernel with local_constant regression and custom bandwidth is exercised in the documented example; other kernel/regression/bandwidth combinations are available but their behaviour is not validated here.
- Performance claims are benchmarked against conditional entropy only.

## Evidence

- [abstract] FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals.: "FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals."
- [abstract] resilience against noise: "It exhibits resilience against noise and adapts well to datasets with varying complexity."
- [abstract] weighting system accounts for uncertainties: "This technique inherently incorporates a weighting system that accounts for measurement uncertainties and facilitates multi-band data analysis."
- [readme] README usage snippet: "best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression("
- [results] frequency grid parameters: "freq_list = np.linspace(0.001,5,50000)"
- [results] kernel configuration: "kernel_type='gaussian',
    regression_type='local_constant',
    bandwidth_method='custom',"
