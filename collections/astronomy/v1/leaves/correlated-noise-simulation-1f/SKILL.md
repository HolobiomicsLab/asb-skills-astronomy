---
name: correlated-noise-simulation-1f
description: Use when you need a signal-plus-correlated-noise simulated polarimeter
  timestream (e.g. LATTE-scale, ~half billion samples) to test a destriping/map-making
  pipeline before real instrument data exist.
when_to_use_negative:
- The goal is destriping already-acquired real data (e.g. the 30 days of B-Machine
  37.5 GHz data available on figshare as ch6_fill_3d.h5) rather than producing a synthetic
  validation dataset.
- Systematic effects other than 1/f correlated noise (gain variations, spin-synchronous
  effects) are the target; the paper notes destriping can be ineffective or even harmful
  for these and they must be dealt with before map-making.
- The noise has knee frequency higher than the scan rate; the paper's destriping approach
  efficiently removes 1/f noise only with knee frequency lower than the scan rate.
edam_operation: http://edamontology.org/operation_0292
tools:
- name: dst
  role: simulation module used to scan the smoothed input sky map with the B-Machine
    scanning strategy and generate the timestream; also the destriper the simulation
    validates
  repo: http://github.com/zonca/dst
- name: h5py
  role: writes the simulated timelines to the single HDF5 file in the column format
    required by dst_io.read_data
- name: HEALPix
  role: pixelization scheme for the input sky map and the map-domain products
- name: Python
  role: implementation language for the simulation, HDF5 I/O, and pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:a2eef43486505599bc99665c9f7756383bf302bcf3ad1a8571a9702386cf1e54
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  - dst
  - h5py
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# correlated-noise-simulation-1f

## Summary

Generate a design-case simulated timestream (~5e8 samples over ~12 months) for validating the dst parallel destriper: scan a smoothed WMAP 7-year Q-band sky map with a B-Machine-style scanning strategy, then add white noise plus 1/f correlated noise using B-Machine channel 6 noise properties, writing an HDF5 file readable by dst_io.read_data.

## When to use

Use when you need a signal-plus-correlated-noise simulated polarimeter timestream (e.g. LATTE-scale, ~half billion samples) to test a destriping/map-making pipeline before real instrument data exist.

## When NOT to use

- The goal is destriping already-acquired real data (e.g. the 30 days of B-Machine 37.5 GHz data available on figshare as ch6_fill_3d.h5) rather than producing a synthetic validation dataset.
- Systematic effects other than 1/f correlated noise (gain variations, spin-synchronous effects) are the target; the paper notes destriping can be ineffective or even harmful for these and they must be dealt with before map-making.
- The noise has knee frequency higher than the scan rate; the paper's destriping approach efficiently removes 1/f noise only with knee frequency lower than the scan rate.

## Inputs

- WMAP 7-year Q-band map (NASA LAMBDA), smoothed by 1 degree gaussian beam
- B-Machine channel 6 noise properties (white noise variance, 1/f knee frequency, slope)
- Scanning strategy parameters: fixed elevation 45 deg, 1 revolution per 70 seconds, ~12 months duration, sunrise/sunset masking

## Outputs

- Single HDF5 timestream file with dtype=[('TIME', '>f8'), ('PHI', '>f8'), ('THETA', '>f8'), ('PSI', '>f8'), ('FLAG', '>i2'), ('TEMP', '>f4'), ('Q', '>f4'), ('U', '>f4')]
- Simulated signal-only timestream (I, q, u plus pointing) before noise addition
- LATTE-scale dataset of about half billion samples (~5e8)

## How to apply

Download the WMAP 7-year Q-band map and smooth it with a 1 degree gaussian beam to reduce noise in map domain. Use the dst simulation module to scan the smoothed map with the B-Machine fixed-elevation (45 deg) strategy at 1 revolution per 70 seconds, extended to about 12 months (~5e8 samples at 33.4 Hz), masking sections near sunrise and sunset that would be contaminated by thermally induced gain variations. Add noise to the signal-only timestream using a white noise variance plus 1/f noise characterized by knee frequency and slope taken from B-Machine channel 6 properties. Write the timelines (I, q, u plus pointing) to a single HDF5 file with the TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U column dtypes required by dst_io.read_data, using h5py. Validate the file exists, opens as HDF5, and contains a sample count on the order of 5e8.

## Related tools

- **dst** (simulation module used to scan the smoothed input sky map with the B-Machine scanning strategy and generate the timestream; also the destriper the simulation validates) — http://github.com/zonca/dst
- **h5py** (writes the simulated timelines to the single HDF5 file in the column format required by dst_io.read_data)
- **HEALPix** (pixelization scheme for the input sky map and the map-domain products)
- **Python** (implementation language for the simulation, HDF5 I/O, and pipeline)

## Examples

```
mpirun -np 3 python dst.py ch6_256.cfg
```

## Evaluation signals

- The output HDF5 file exists and opens with h5py.
- Sample count is on the order of 5e8 (about half a billion samples for 12 months, consistent with the upcoming LATTE dataset expectation).
- Dataset dtype matches exactly: [('TIME', '>f8'), ('PHI', '>f8'), ('THETA', '>f8'), ('PSI', '>f8'), ('FLAG', '>i2'), ('TEMP', '>f4'), ('Q', '>f4'), ('U', '>f4')].
- The file is readable by dst_io.read_data and can be destriped with the example config (e.g. python dst.py ch6_256.cfg).
- Sections near sunrise and sunset (thermally contaminated) are masked out in the FLAG column.

## Limitations

- Simulated noise properties come from B-Machine channel 6; extrapolation to LATTE's 8 GHz channel is an assumption.
- The simulation covers only the design case of signal and correlated noise; additional systematic effects must be modeled separately.
- The destriper this simulates for requires a scanning strategy with frequent crossing points to be effective.

## Evidence

- [methods] The input signal was based on WMAP 7 years Q band, smoothed by 1◦ in order to reduce the noise in map domain: [evidence span withheld — anchor status: not_found]
- [intro] knee frequency vs scan rate: [evidence span withheld — anchor status: not_found]
- [methods] sunrise/sunset masking: [evidence span withheld — anchor status: not_found]
