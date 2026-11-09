---
name: nonparametric-kernel-regression-period-finding
description: Use when you need to recover the dominant frequency/period of a sparsely
  sampled astronomical light curve (even ~10 observations) from time/magnitude/uncertainty
  arrays, especially when noise robustness or multi-band handling matters.
when_to_use_negative:
- Input is already a feature table or aggregated statistics rather than raw time-series
  observations with times, magnitudes, and uncertainties.
- The target signal requires periodogram-style Fourier power (e.g. strictly coherent
  sinusoid detection) rather than nonparametric folded-light-curve regression.
- You have no per-observation uncertainty estimates and cannot justify the uncertainty
  weighting the method relies on.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: FINKER
  role: 'Core implementation: parallel_nonparametric_kernel_regression performs the
    folded-light-curve Gaussian-kernel local-constant regression frequency search
    with uncertainty weighting'
  repo: github.com/FiorenSt/FINKER
- name: NumPy
  role: Builds the trial frequency grid (np.linspace(0.001, 5, 50000)) and supplies
    the t_observed/y_observed/uncertainties arrays
- name: Python
  role: Execution environment; FINKER is pip-installable and imported via 'from FINKER.utils_FINKER
    import *'
  repo: github.com/FiorenSt/FINKER
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:ceaa52fb341dfc7b26ddec52960745db045e13fe53fe84fda27b1de1b7cc2c9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# nonparametric-kernel-regression-period-finding

## Summary

Period/frequency identification for astronomical time series by applying nonparametric kernel regression (Gaussian kernel, local_constant fit) to data folded at trial frequencies, then selecting the frequency that minimises squared residuals. Implemented by the FINKER package, which weights observations by measurement uncertainties.

## When to use

Use when you need to recover the dominant frequency/period of a sparsely sampled astronomical light curve (even ~10 observations) from time/magnitude/uncertainty arrays, especially when noise robustness or multi-band handling matters.

## When NOT to use

- Input is already a feature table or aggregated statistics rather than raw time-series observations with times, magnitudes, and uncertainties.
- The target signal requires periodogram-style Fourier power (e.g. strictly coherent sinusoid detection) rather than nonparametric folded-light-curve regression.
- You have no per-observation uncertainty estimates and cannot justify the uncertainty weighting the method relies on.

## Inputs

- t_observed: numpy array of observation times
- y_observed: numpy array of magnitudes/fluxes
- uncertainties: numpy array of per-observation measurement uncertainties (used as regression weights)
- freq_list: numpy array of trial frequencies (e.g. np.linspace(0.001, 5, 50000))
- Optional: synthetic light curve from FINKER.utils_SYNTHETIC_LIGHTCURVES (n_points=100, time=10, freq_primary, snr_primary=7, baseline_magnitude=17.0)

## Outputs

- best_freq: the frequency minimising squared residuals across the search grid
- freq_err: estimated uncertainty on best_freq
- result_dict: table of (frequency, squared residual) pairs, savable as result_dict.csv for a squared-residual vs trial-frequency scatter plot

## How to apply

Instantiate a FINKER object and build a dense trial frequency grid with NumPy (the paper's synthetic test uses freq_list = np.linspace(0.001, 5, 50000)). Call finker.parallel_nonparametric_kernel_regression passing t_observed, y_observed, uncertainties and the frequency grid, configured with kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', and n_jobs=-2 for the parallel search. At each trial frequency the observations are folded and fit with a Gaussian-kernel local-constant regression weighted by measurement uncertainties; squared residuals are accumulated per frequency into result_dict. Select best_freq as the grid frequency minimising squared residuals, and record the returned freq_err. Validate by checking that the minimum of the squared-residual-vs-frequency curve coincides with any known/injected frequency within grid resolution.

## Related tools

- **FINKER** (Core implementation: parallel_nonparametric_kernel_regression performs the folded-light-curve Gaussian-kernel local-constant regression frequency search with uncertainty weighting) — github.com/FiorenSt/FINKER
- **NumPy** (Builds the trial frequency grid (np.linspace(0.001, 5, 50000)) and supplies the t_observed/y_observed/uncertainties arrays)
- **Python** (Execution environment; FINKER is pip-installable and imported via 'from FINKER.utils_FINKER import *') — github.com/FiorenSt/FINKER

## Examples

```
from FINKER.utils_FINKER import *; finker = FINKER(); best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(t_observed=t, y_observed=y, uncertainties=err, freq_list=np.linspace(0.001,5,50000), show_plot=False, kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', n_jobs=-2)
```

## Evaluation signals

- The minimum of the squared-residual vs frequency curve coincides with the known/injected true frequency (0.6 in the paper's synthetic test with n_points=100, time=10, snr_primary=7) within the grid resolution.
- best_freq is returned together with a finite freq_err from the FINKER call.
- result_dict contains one (frequency, squared residual) entry per grid point (50000 rows for the paper's grid) and can be saved as result_dict.csv.
- The search runs with the documented configuration: kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom'; deviating bandwidth choices (Silverman, Scott, Adaptive) change the curve and should be flagged.
- Folded light curve at best_freq shows a coherent, low-residual fit consistent with the uncertainty-weighted local-constant model.

## Limitations

- Bandwidth choice matters: the paper's example uses bandwidth_method='custom' (a custom fixed bandwidth for computational efficiency); Silverman's rule, Scott's rule, and Adaptive bandwidth are available but not benchmarked in the shown test.
- The article's discussion section is empty in the extracted context ('No changelog found.' placeholder), so limitations and reproducibility gaps are not documented there.
- Performance was compared against conditional entropy; relative accuracy/robustness claims rest on the authors' benchmarks (superior performance, fewer observations needed, resilience to noise).

## Evidence

- [abstract] FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals.: "FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals."
- [results] gaussian kernel with local_constant regression and custom bandwidth: "kernel_type='gaussian',
    regression_type='local_constant',
    bandwidth_method='custom',"
- [results] frequency grid definition: "freq_list = np.linspace(0.001,5,50000)"
- [readme] README usage snippet: "best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression("
- [abstract] weighting by uncertainties: "This technique inherently incorporates a weighting system that accounts for measurement uncertainties and facilitates multi-band data analysis."
