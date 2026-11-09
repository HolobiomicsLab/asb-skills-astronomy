---
name: astronomical-time-series-phase-folding
description: Use when you have sparse astronomical light curves (time, magnitude,
  measurement uncertainties) with an unknown periodicity, and need to identify the
  true frequency by folding at trial frequencies on a grid (e.g. np.linspace(0.001,
  5, 50000)) and minimising a regression-fit statistic.
when_to_use_negative:
- The data are non-periodic or the goal is trend/aperiodic variability analysis rather
  than frequency identification.
- A parametric model of the periodic signal is already known — kernel regression folding
  is a nonparametric alternative, not a model-fitting replacement.
- You need frequency identification without measurement uncertainties or with no foldable
  phase information; the weighting system is built around measurement uncertainties.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: FINKER
  role: Performs nonparametric kernel regression on phase-folded data at trial frequencies
    (parallel_nonparametric_kernel_regression) and selects the optimal frequency by
    minimising squared residuals.
  repo: github.com/FiorenSt/FINKER
- name: NumPy
  role: Builds the trial frequency grid (np.linspace(0.001, 5, 50000)) and supplies
    observation arrays.
- name: Python
  role: Runs the FINKER workflow, folding, fitting, and export of result_dict.csv
    and figures.
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:42ac7e08d3da349d8454a9422bef06c1e8de38b9f738b6310f6fe1e5cc234865
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# astronomical-time-series-phase-folding

## Summary

Phase-fold astronomical time series at trial frequencies and fit nonparametric kernel regression (FINKER) to the folded data, selecting the optimal frequency by minimising squared residuals — the core frequency-identification workflow of the FINKER paper.

## When to use

Use when you have sparse astronomical light curves (time, magnitude, measurement uncertainties) with an unknown periodicity, and need to identify the true frequency by folding at trial frequencies on a grid (e.g. np.linspace(0.001, 5, 50000)) and minimising a regression-fit statistic.

## When NOT to use

- The data are non-periodic or the goal is trend/aperiodic variability analysis rather than frequency identification.
- A parametric model of the periodic signal is already known — kernel regression folding is a nonparametric alternative, not a model-fitting replacement.
- You need frequency identification without measurement uncertainties or with no foldable phase information; the weighting system is built around measurement uncertainties.

## Inputs

- Light curve time series: t_observed, y_observed and uncertainties as NumPy arrays (magnitudes with measurement errors)
- Trial frequency grid, e.g. np.linspace(0.001, 5, 50000)
- Optional multi-band observations (FINKER supports multiband processing)

## Outputs

- best_freq (optimal frequency estimate) and freq_err (frequency error) from the parallel search
- result_dict table of (frequency, squared residual) pairs, exportable as result_dict.csv
- Squared-residual versus trial-frequency scatter plot
- Folded light curve with smoothed kernel-regression fit at the estimated frequency (folded_light_curve_smoothed_fit.png), with phase, y_smoothed, bw and squared_residual returned

## How to apply

Generate or load the light curve (e.g. via generator.generate_synthetic_light_curve with n_points=100, time=10, freq_primary=0.6, snr_primary=7, baseline_magnitude=17.0), then build a trial frequency grid with NumPy as freq_list = np.linspace(0.001, 5, 50000). Call finker.parallel_nonparametric_kernel_regression passing t_observed, y_observed and uncertainties, with kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom' and n_jobs=-2; at each trial frequency the data are phase-folded and a Gaussian-kernel local-constant fit is applied, weighting observations by their measurement uncertainties. Collect squared residuals per frequency into result_dict, pick the grid frequency that minimises them to obtain best_freq and freq_err, and optionally refit FINKER's nonparametric_kernel_regression at best_freq (show_plot=True) to render the folded light curve with the smoothed fit. Validate by checking that the minimum of the squared-residual curve coincides with the known/injected frequency within grid resolution, and optionally benchmark against the conditional entropy period-finding algorithm on the same grid.

## Related tools

- **FINKER** (Performs nonparametric kernel regression on phase-folded data at trial frequencies (parallel_nonparametric_kernel_regression) and selects the optimal frequency by minimising squared residuals.) — github.com/FiorenSt/FINKER
- **NumPy** (Builds the trial frequency grid (np.linspace(0.001, 5, 50000)) and supplies observation arrays.)
- **Python** (Runs the FINKER workflow, folding, fitting, and export of result_dict.csv and figures.)

## Examples

```
from FINKER.utils_FINKER import *; finker = FINKER(); best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(t_observed=t, y_observed=y, uncertainties=u, freq_list=np.linspace(0.001,5,50000), kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', n_jobs=-2)
```

## Evaluation signals

- The minimum of the squared-residual vs trial-frequency curve coincides with the known/injected true frequency (0.6 in the synthetic test) within the grid resolution.
- best_freq falls within the searched frequency range (0.001–5) and freq_err is returned alongside it.
- result_dict.csv exists with (frequency, squared residual) rows covering all 50000 trial frequencies.
- folded_light_curve_smoothed_fit.png is a rendered figure in which the smoothed fit traces the periodic pattern of the folded light curve points.
- A baseline estimate from the conditional entropy algorithm on the same grid is comparable to the FINKER estimate for the same curve.

## Limitations

- The synthetic test used only n_points=100 observations over time=10 with snr_primary=7; performance on much denser or higher-SNR data was not demonstrated in the extracted sections.
- No discussion-section content is available, so limitations, missing artifacts, or reproducibility gaps cannot be extracted from the article.
- The custom bandwidth method is a fixed bandwidth chosen for computational efficiency; other options (Silverman's rule, Scott's rule, Adaptive bandwidth) exist but were not exercised in the demonstrated workflow.

## Evidence

- [abstract] FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals.: "FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals."
- [results] Gaussian kernel, local_constant regression, custom bandwidth: "kernel_type='gaussian',
    regression_type='local_constant',
    bandwidth_method='custom',"
- [results] Frequency grid definition: "freq_list = np.linspace(0.001,5,50000)"
- [abstract] Uncertainty weighting claim: "This technique inherently incorporates a weighting system that accounts for measurement uncertainties and facilitates multi-band data analysis."
- [readme] README usage snippet: "best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression("
