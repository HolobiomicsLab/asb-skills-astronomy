---
name: radio-continuum-flux-frequency-conversion
description: Use when the available catalogue has 1.4 GHz (or other non-150 MHz) radio
  continuum fluxes but the target classifier expects radiototal/radiopeak at 150 MHz,
  and native LOFAR 150 MHz data are unavailable.
when_to_use_negative:
- Native 150 MHz LOFAR radio data are already available — the README warns misclassifications
  can occur if 1.4 GHz data are used instead of 150 MHz, and conversion introduces
  an avoidable ~4%/~3% performance penalty.
- You need a calibrated probability of AGN nature — the raw output column does not
  correspond to a likelihood that a source is an AGN, so it should not be interpreted
  probabilistically in either configuration.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MBASC
  role: Classifier run on the frequency-converted catalogue to produce AGN/SFG predictions
  repo: github.com/Jesper-Karsten/MBASC
- name: Python 3
  role: Runtime for the external spectral-index conversion script and for MBASC.py
- name: pandas
  role: Column renaming (radiototal, radiopeak) and unit conversion of the input CSV
- name: sklearn
  role: Computing precision and recall for SFGs and AGNs to measure the conversion-induced
    degradation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:8f0728a934b0d51c94b935645e8b9d60155034c8fea03cf7a846f1885cf81ba3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MBASC
  - Python
  - Python 3
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# radio-continuum-flux-frequency-conversion

## Summary

Out-of-tool procedure for scaling 1.4 GHz radio flux densities to the 150 MHz values MBASC expects, via a radio spectral-index conversion applied before classification. Quantifies the accuracy penalty incurred by this workaround versus native 150 MHz data.

## When to use

Use when the available catalogue has 1.4 GHz (or other non-150 MHz) radio continuum fluxes but the target classifier expects radiototal/radiopeak at 150 MHz, and native LOFAR 150 MHz data are unavailable.

## When NOT to use

- Native 150 MHz LOFAR radio data are already available — the README warns misclassifications can occur if 1.4 GHz data are used instead of 150 MHz, and conversion introduces an avoidable ~4%/~3% performance penalty.
- You need a calibrated probability of AGN nature — the raw output column does not correspond to a likelihood that a source is an AGN, so it should not be interpreted probabilistically in either configuration.

## Inputs

- Catalogue (CSV) with 1.4 GHz radio continuum flux densities to be scaled to 150 MHz
- Reference labels (AGN/SFG) for computing precision and recall with sklearn
- Optional: native 150 MHz LOFAR baseline results for delta comparison

## Outputs

- Converted catalogue with 150 MHz radiototal and radiopeak columns in μJy
- output/results.csv with id, output (raw LightGBM fraction 0–1), and binary AGN column (1 = AGN, 0 = SFG)
- Precision/recall deltas versus the native 150 MHz baseline (expected ~4% precision drop, ~3% recall drop)

## How to apply

Apply a radio spectral-index scaling outside the MBASC tool to convert 1.4 GHz flux densities to the 150 MHz values the model was trained on (per the README, users 'will have to convert it yourself'). Then convert every flux column (except redshift) to μJy and rename the radio columns to radiototal and radiopeak so they match MBASC's required headers. Run the classifier (python MBASC.py [path/to/file.csv]) on the converted catalogue, read output/results.csv, and round the raw LightGBM fraction (0–1) into the binary AGN column. Finally, benchmark precision/recall for SFGs and AGNs with sklearn against the native-150-MHz baseline (SFG precision 0.92±0.01, recall 0.87±0.02; AGN precision 0.87±0.02, recall 0.78±0.02); expect a degradation of ~4% in precision and ~3% in recall from the conversion.

## Related tools

- **MBASC** (Classifier run on the frequency-converted catalogue to produce AGN/SFG predictions) — github.com/Jesper-Karsten/MBASC
- **Python 3** (Runtime for the external spectral-index conversion script and for MBASC.py)
- **pandas** (Column renaming (radiototal, radiopeak) and unit conversion of the input CSV)
- **sklearn** (Computing precision and recall for SFGs and AGNs to measure the conversion-induced degradation)

## Examples

```
python MBASC.py converted_150MHz_catalogue.csv
```

## Evaluation signals

- Converted-input run shows a precision drop of approximately 4% and a recall drop of approximately 3% relative to the native 150 MHz baseline, as reported in the abstract.
- All flux columns in the converted catalogue are in μJy and radio columns carry the exact header names radiototal and radiopeak.
- output/results.csv contains exactly the columns id, output, and AGN, in the same row order as the input file.
- Raw output values are fractions between 0 and 1; the AGN column is their rounding (1 = AGN, 0 = SFG).
- Reference baseline reproduced: SFG precision 0.92±0.01, recall 0.87±0.02; AGN precision 0.87±0.02, recall 0.78±0.02.

## Limitations

- The conversion must be performed by the user outside MBASC; it is not implemented in the tool (though 'this might be added to the model itself later').
- Converted 1.4 GHz data systematically degrade performance (~4% precision, ~3% recall) versus native 150 MHz data.
- MBASC expects 150 MHz radio data specifically; using 1.4 GHz or other wavebands without conversion is listed as a troubleshooting cause of systematic misclassifications.
- Cross-matched radio data can be problematic — wrongly cross-matched radio columns can yield bad results even after correct frequency conversion.

## Evidence

- [abstract] Converting 1.4 GHz to 150 MHz radio data reduces the performance by ~4% in precision and ~3% in recall.: "Converting 1.4 GHz to 150 MHz radio data reduces the performance by ~4% in precision and ~3% in recall."
- [readme] Yes, but you will have to convert it yourself: "Yes, but you will have to convert it yourself (this might be added to the model itself later). See the corresponding paper for performance with generated 150 MHz data"
- [readme] All units (except redshift) must be converted to μJy.: "All units (except redshift) must be converted to μJy."
- [readme] Ensure that you used 150 MHz radio data, not 1.4 GHz or some other waveband.: "Ensure that you used 150 MHz radio data, not 1.4 GHz or some other waveband."
- [methods] This value is between 0 and 1 and is simply rounded to result in the AGN column.: "This value is between 0 and 1 and is simply rounded to result in the AGN column."
