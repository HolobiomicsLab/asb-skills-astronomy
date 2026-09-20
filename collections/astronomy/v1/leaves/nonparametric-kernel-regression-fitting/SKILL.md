---
name: nonparametric-kernel-regression-fitting
description: Use when a magnitude time series (light curve) with per-point uncertainties
  must have its periodicity detected/recovered, especially sparse data (down to ~10
  observations) or multi-band data, by fitting nonparametric kernel regression on
  phase-folded datasets.
when_to_use_negative:
- The goal is exploratory plotting of raw, unfolded photometry rather than frequency
  estimation or a phase-folded smoothed fit
- A strictly parametric/analytic periodogram (e.g. conditional entropy) is mandated
  for the benchmark; FINKER is the nonparametric alternative being compared against
  it
- The dataset is not a time series with per-point uncertainties, so the uncertainty-weighted
  kernel regression has no meaningful input
edam_operation: http://edamontology.org/operation_2945
tools:
- name: FINKER
  role: Provides parallel_nonparametric_kernel_regression and nonparametric_kernel_regression
    (Gaussian kernel, local_constant regression, custom bandwidth) used to search
    frequencies and produce the folded smoothed fit
  repo: github.com/FiorenSt/FINKER
- name: Python
  role: Runtime for the FINKER package; numpy arrays supply t_observed, y_observed,
    uncertainties and the frequency grid
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202348848
    title: 'FINKER: Frequency Identification through Nonparametric KErnel Regression
      in astronomical time series'
  source_content_hash: sha256:d4c52dbee18c3aa68154d131b0ab4bb47654af098fce31b10aa5ac5893df9606
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FINKER
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nonparametric-kernel-regression-fitting

## Summary

Frequency identification for astronomical time series via FINKER's nonparametric kernel regression: datasets are folded at trial frequencies, a Gaussian-kernel local-constant fit with measurement-uncertainty weighting is computed, and the optimal frequency is selected by minimising squared residuals.

## When to use

Use when a magnitude time series (light curve) with per-point uncertainties must have its periodicity detected/recovered, especially sparse data (down to ~10 observations) or multi-band data, by fitting nonparametric kernel regression on phase-folded datasets.

## When NOT to use

- The goal is exploratory plotting of raw, unfolded photometry rather than frequency estimation or a phase-folded smoothed fit
- A strictly parametric/analytic periodogram (e.g. conditional entropy) is mandated for the benchmark; FINKER is the nonparametric alternative being compared against it
- The dataset is not a time series with per-point uncertainties, so the uncertainty-weighted kernel regression has no meaningful input

## Inputs

- t_observed: numpy array of observation times
- y_observed: numpy array of magnitudes (e.g. baseline_magnitude=17.0 light curve)
- uncertainties: numpy array of per-point measurement uncertainties used as kernel weights
- freq_list: numpy array of trial frequencies, e.g. np.linspace(0.001, 5, 50000)

## Outputs

- best_freq: optimal frequency selected by minimising squared residuals (e.g. ~0.6)
- freq_err: reported frequency uncertainty
- result_dict: dictionary containing best_freq, freq_err and the selected kernel bandwidth (bw)
- phase, y_smoothed, bw, squared_residual from the single-frequency nonparametric_kernel_regression smoothed-fit call

## How to apply

Instantiate FINKER (pip install FINKER; from FINKER.utils_FINKER import *) and call parallel_nonparametric_kernel_regression on t_observed, y_observed and uncertainties over a frequency grid (e.g. np.linspace(0.001, 5, 50000)) with kernel_type='gaussian', regression_type='local_constant', and bandwidth_method='custom' (Silverman, Scott, or Adaptive bandwidths are also available). The method folds the dataset at each trial frequency, applies local constant (or local linear) kernel regression with a weighting system accounting for measurement uncertainties, and returns best_freq, freq_err, and a result_dict containing the selected kernel bandwidth. n_jobs controls parallelism (n_jobs=-2 in the reference example). For a smoothed-fit diagnostic, call nonparametric_kernel_regression at best_freq alone to obtain phase, y_smoothed, bw, and squared_residual, and plot with show_plot=True; the smoothed fit should trace the periodic pattern of the folded points. Validate by checking that best_freq recovers the true frequency (e.g. 0.6 in the synthetic test) within the reported freq_err.

## Related tools

- **FINKER** (Provides parallel_nonparametric_kernel_regression and nonparametric_kernel_regression (Gaussian kernel, local_constant regression, custom bandwidth) used to search frequencies and produce the folded smoothed fit) — github.com/FiorenSt/FINKER
- **Python** (Runtime for the FINKER package; numpy arrays supply t_observed, y_observed, uncertainties and the frequency grid)

## Examples

```
best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression(t_observed=t_observed, y_observed=y_observed, uncertainties=uncertainties, freq_list=freq, show_plot=False, kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom', n_jobs=-2)
```

## Evaluation signals

- best_freq recovers the known true frequency (0.6 in the synthetic test with n_points=100, time=10, snr_primary=7) within the reported freq_err
- Returned result_dict contains the fields best_freq, freq_err and bandwidth; a JSON record (finker_frequency_search_results.json) must include all three keys
- The smoothed-fit call returns phase, y_smoothed, bw and squared_residual; squared_residual is minimal near best_freq on the grid
- folded_light_curve_smoothed_fit.png exists and its smoothed kernel-regression fit traces the periodic pattern of the folded light curve points
- Keyword arguments match the reference configuration: kernel_type='gaussian', regression_type='local_constant', bandwidth_method='custom'

## Limitations

- Bandwidth choice affects the fit: 'custom' fixed bandwidth is used for computational efficiency, with Silverman's rule, Scott's rule and Adaptive bandwidth as alternatives
- Robustness/resilience to noise is claimed but demonstrated only on synthetic and comparison tests against conditional entropy; no discussion-section limitations were extracted (the discussion contained only a 'No changelog found.' placeholder)
- Requires per-point measurement uncertainties for the built-in weighting system; n_jobs setting (e.g. -2) affects parallel search resource use

## Evidence

- [abstract] FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals.: "FINKER uses nonparametric kernel regression on folded datasets at different frequencies, selecting the optimal frequency by minimising squared residuals."
- [abstract] This technique inherently incorporates a weighting system that accounts for measurement uncertainties and facilitates multi-band data analysis.: "This technique inherently incorporates a weighting system that accounts for measurement uncertainties and facilitates multi-band data analysis."
- [readme] README usage snippet showing the parallel search call and keyword arguments.: "best_freq, freq_err, result_dict = finker.parallel_nonparametric_kernel_regression("
- [readme] README on bandwidth options and local regression.: [evidence span withheld — anchor status: external_doc]
- [results] Frequency grid used in the article's synthetic test.: "freq_list = np.linspace(0.001,5,50000)"
