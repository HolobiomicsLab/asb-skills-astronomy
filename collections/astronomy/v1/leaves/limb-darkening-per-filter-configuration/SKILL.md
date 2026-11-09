---
name: limb-darkening-per-filter-configuration
description: Use when fitting multiple transit light curves observed under different
  filters, instruments, or weather/limb-darkening conditions simultaneously, and you
  must let limb darkening vary per curve while sharing system parameters across curves.
when_to_use_negative:
- Only a single light curve in a single filter is available — run TAP's single-curve
  mode instead; the multicurve locking machinery adds no benefit
- The light curves must be phase folded and rebinned for a coarse aggregation analysis
  — TAP is explicitly designed to work without phase folding or rebinning
- The noise is known to be purely uncorrelated and a classic chi-squared treatment
  is adequate — the wavelet-based likelihood machinery would be unnecessary overhead
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Transit Analysis Package (TAP)
  role: Runs the multicurve MCMC with the parameter-locking matrix controlling which
    parameters evolve together vs. per curve
  repo: https://github.com/zgazak/TAP
- name: IDL
  role: Language/runtime of the TAP GUI used to set up per-filter limb-darkening and
    locked system parameters
- name: EXOFAST
  role: Provides the analytic Mandel and Agol (2002) transit model evaluated by TAP
    during MCMC
- name: Kepler mission
  role: Source of long-cadence photometry requiring long-integration handling in the
    multicurve fit
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:be57cf27c76291f841c9f5377be9e5e2f8e2b0170eedf8b01d1ea9dcabc76973
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Kepler mission
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# limb-darkening-per-filter-configuration

## Summary

Configure TAP's multicurve parameter-locking matrix so limb-darkening coefficients evolve independently per light curve (per filter/instrument) while system parameters (P, a/Rs, Rp/Rs, i, Tmid) are locked to evolve together across all curves during MCMC, avoiding the need to phase fold or rebin.

## When to use

Use when fitting multiple transit light curves observed under different filters, instruments, or weather/limb-darkening conditions simultaneously, and you must let limb darkening vary per curve while sharing system parameters across curves.

## When NOT to use

- Only a single light curve in a single filter is available — run TAP's single-curve mode instead; the multicurve locking machinery adds no benefit
- The light curves must be phase folded and rebinned for a coarse aggregation analysis — TAP is explicitly designed to work without phase folding or rebinning
- The noise is known to be purely uncorrelated and a classic chi-squared treatment is adequate — the wavelet-based likelihood machinery would be unnecessary overhead

## Inputs

- Multiple normalized transit light curves of a single target (e.g., Kepler long-cadence photometry or ground-based multi-filter photometry), not phase folded or rebinned
- TAP multicurve configuration specifying the shared vs. per-curve parameter-locking matrix (limb darkening set free per curve; P, a/Rs, Rp/Rs, i, Tmid locked)
- Filter/instrument metadata per curve to justify independent limb-darkening treatment

## Outputs

- Per-transit mid-transit times (Tmid) with posterior uncertainties
- Combined (locked) system parameters (P, a/Rs, Rp/Rs, i) with posterior uncertainties
- MCMC chains / posterior parameter table (e.g., multitransit_results.csv)

## How to apply

In TAP (IDL, using the EXOFAST implementation of the Mandel and Agol 2002 model), build the multicurve parameter matrix so that system parameters P, a/Rs, Rp/Rs, i, and Tmid are flagged to evolve together (locked) across the N light curves, while limb-darkening coefficients remain free and independent per curve, along with per-curve wavelet noise and trend parameters. This is justified because TAP supports 'simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions' — different filters legitimately change limb darkening, so locking it would bias the fit. Normalize each transit segment without phase folding or rebinning, and for long-integration Kepler data enable fine-temporal-scale model computation integrated to the observed cadence. Run the MCMC with the Carter and Winn (2009) wavelet-based likelihood so uncorrelated and correlated noise are parameterized, then extract per-transit Tmid values and the combined locked system parameters with posterior uncertainties.

## Related tools

- **Transit Analysis Package (TAP)** (Runs the multicurve MCMC with the parameter-locking matrix controlling which parameters evolve together vs. per curve) — https://github.com/zgazak/TAP
- **IDL** (Language/runtime of the TAP GUI used to set up per-filter limb-darkening and locked system parameters)
- **EXOFAST** (Provides the analytic Mandel and Agol (2002) transit model evaluated by TAP during MCMC)
- **Kepler mission** (Source of long-cadence photometry requiring long-integration handling in the multicurve fit)

## Evaluation signals

- Per-curve limb-darkening posteriors differ across curves (as expected for different filters), while locked system parameters show a single combined posterior across all curves
- Combined-parameter uncertainties from the multicurve lock are no larger than the uncertainties from independently refitting each light curve in single-curve mode
- The analysis was performed on un-folded, un-rebinned normalized transit segments — no phase folding artifacts present
- Wavelet-likelihood noise parameters (sigma_w, sigma_r or equivalent) were fit per curve rather than shared, and posterior parameter values agree with prior published results for the target (e.g., TrES-1b, WASP-4b, Kepler-4b to 8b style cross-checks)

## Limitations

- Derived transit parameters can vary between fitting methods; confidence depends critically on the treatment of noise, stellar limb darkening, and cadence — per-filter limb-darkening configuration only mitigates this if filters genuinely differ
- Long-integration distortions (most Kepler light curves) require the model to be computed on a finer temporal scale and integrated to the observed cadence; omitting this biases the fit
- Verification against single-curve refits is expected behavior of the algorithm, so a combined fit that widens uncertainties signals misconfiguration

## Evidence

- [intro] simultaneous multi-curve analysis without phase folding: "TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
- [abstract] multi-condition transits: "The software is able to simultaneously analyze multiple transits observed in different conditions (instrument, filter, weather, etc.)."
- [intro] wavelet likelihood for uncertainties: [evidence span withheld — anchor status: not_found]
- [intro] long-integration Kepler handling: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
