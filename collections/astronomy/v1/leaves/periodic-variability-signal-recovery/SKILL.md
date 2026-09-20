---
name: periodic-variability-signal-recovery
description: Use when you have a sparsely sampled, noisy time series (e.g. magnitudes
  with uncertainties from a synthetic or observed light curve) and need to identify
  a periodic frequency and visualize the folded, smoothed light curve.
when_to_use_negative:
- The data are not time-stamped observations with uncertainties (e.g. already folded,
  or a feature table) — folding and frequency search do not apply
- You need a different period-finding paradigm such as conditional entropy; FINKER
  is benchmarked against conditional entropy rather than a drop-in replacement with
  identical semantics
- Only a single fixed frequency matters and no search over a frequency list is required
tools:
- name: FINKER
  role: Runs the parallel nonparametric kernel-regression frequency search and the
    folded-light-curve smoothed fit with show_plot=True
  repo: github.com/FiorenSt/FINKER
- name: Python
  role: Scripting environment used to call FINKER utilities and generate/save the
    figure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:5d007be2e27e7c87e2a77bccd94b90722a1cc335995efe82c0e6589901abdf9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# periodic-variability-signal-recovery

## Summary

Recover the periodic signal of an astronomical time series with FINKER: run a parallel nonparametric kernel-regression frequency search (Gaussian kernel, local_constant regression, custom bandwidth), then fold the light curve at the best frequency and render a smoothed-fit diagnostic plot.

## When to use

Use when you have a sparsely sampled, noisy time series (e.g. magnitudes with uncertainties from a synthetic or observed light curve) and need to identify a periodic frequency and visualize the folded, smoothed light curve.

## When NOT to use

- The data are not time-stamped observations with uncertainties (e.g. already folded, or a feature table) — folding and frequency search do not apply
- You need a different period-finding paradigm such as conditional entropy; FINKER is benchmarked against conditional entropy rather than a drop-in replacement with identical semantics
- Only a single fixed frequency matters and no search over a frequency list is required

## Inputs

- Time series arrays: t_observed, y_observed, uncertainties (numpy arrays)
- Frequency search grid (e.g. np.linspace(0.001, 5, 50000))
- Synthetic light curve parameters (n_points, time, freq_primary, snr_primary, baseline_magnitude)

## Outputs

- best_freq, freq_err, and result_dict from the parallel kernel-regression search
- Folded phase, y_smoothed fit, bandwidth bw, and squared_residual from the nonparametric kernel regression at the estimated frequency
- Rendered diagnostic figure folded_light_curve_smoothed_fit.png

## How to apply

Generate or load the light curve (article test case: n_points=100, time=10, snr_primary=7, baseline_magnitude=17.0, true_freq=0.6). Build a search grid such as np.linspace(0.001,5,50000) and call finker.parallel_nonparametric_kernel_regression with kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', and n_jobs=-2, passing t_observed, y_observed, uncertainties as numpy arrays. FINKER folds the data at each trial frequency, fits the nonparametric kernel regression, and selects the frequency minimising squared residuals, returning best_freq, freq_err, and result_dict. Refit the nonparametric kernel regression on the folded points at best_freq, then call the plotting routine with show_plot=True to overlay the y_smoothed fit on the folded data and save folded_light_curve_smoothed_fit.png.

## Related tools

- **FINKER** (Runs the parallel nonparametric kernel-regression frequency search and the folded-light-curve smoothed fit with show_plot=True) — github.com/FiorenSt/FINKER
- **Python** (Scripting environment used to call FINKER utilities and generate/save the figure)

## Examples

```
from FINKER.utils_FINKER import *; finker = FINKER(); best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(t_observed=t_observed, y_observed=y_observed, uncertainties=uncertainties, freq_list=freq, show_plot=False, kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', n_jobs=-2)
```

## Evaluation signals

- folded_light_curve_smoothed_fit.png exists and the smoothed fit traces the periodic pattern of the folded points
- best_freq returned by the search matches the injected true frequency (0.6 in the article's synthetic test) within freq_err
- The chosen frequency is the one that minimises squared residuals across the freq_list grid
- Returned objects include phase, y_smoothed, bw, and squared_residual from the kernel regression call

## Limitations

- The article's discussion section is empty ('No changelog found.' placeholder), so limitations, reproducibility gaps, and missing artifacts cannot be enumerated from the paper
- bandwidth_method='custom' uses a fixed bandwidth for computational efficiency; alternatives (Silverman, Scott, adaptive) exist and may be needed for other datasets
- Frequentist search over a large grid (50000 frequencies) may be computationally expensive; the README example uses n_jobs=-2 for parallelism

## Evidence

- [abstract] FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals.: "FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals."
- [results] kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom',: "kernel_type='gaussian',
    regression_type='local_constant',
    bandwidth_method='custom',"
- [readme] best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(: "best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression("
- [results] The frequency search was performed over a frequency list from 0.001 to 5 with 50000 points: "freq_list = np.linspace(0.001,5,50000)"
