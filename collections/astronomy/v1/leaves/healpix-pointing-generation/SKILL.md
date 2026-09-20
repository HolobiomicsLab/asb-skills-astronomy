---
name: healpix-pointing-generation
description: Use when simulating a polarimeter timestream that requires a realistic
  scanning strategy (fixed elevation, sky rotation) converted into HEALPix PHI/THETA/PSI
  pointing columns for map-making or destriping.
when_to_use_negative:
- Input is already a calibrated timestream with pointing columns (PHI/THETA/PSI) —
  no pointing generation is needed
- The analysis concerns real B-Machine/LATTE data where pointing is reconstructed
  from raw instrument data rather than simulated
- The scanning strategy differs fundamentally from the B-Machine model (e.g. different
  elevation, frame, or rotation rate) and the fixed 45-degree/70-second parameters
  would be invalid
edam_operation: http://edamontology.org/operation_2429
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: HEALPix
  role: Pixelizes the sky and defines the PHI/THETA pointing grid at Nside 512 used
    for the scanning timeline
- name: dst
  role: Consumes the generated pointing columns via dst_io.read_data as part of the
    destriping input HDF5 file
  repo: http://github.com/zonca/dst
- name: Python
  role: Implementation language for computing the pointing timeline and writing the
    HDF5 output
- name: B-Machine
  role: Instrument whose scanning geometry (45 deg elevation, 70 s revolution, 33.4
    Hz sampling) defines the pointing model parameters
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:edf2f7821761ee2914111ae12c1f62e05b639d226d0e7893e69864a14181436d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  - dst
  edam_operation: http://edamontology.org/operation_2429
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# healpix-pointing-generation

## Summary

Generates the pointing/scanning timeline for the B-Machine ground-based CMB polarimeter in HEALPix sky coordinates, producing the PHI/THETA/PSI columns needed to scan a sky map into a simulated timestream for destriping tests.

## When to use

Use when simulating a polarimeter timestream that requires a realistic scanning strategy (fixed elevation, sky rotation) converted into HEALPix PHI/THETA/PSI pointing columns for map-making or destriping.

## When NOT to use

- Input is already a calibrated timestream with pointing columns (PHI/THETA/PSI) — no pointing generation is needed
- The analysis concerns real B-Machine/LATTE data where pointing is reconstructed from raw instrument data rather than simulated
- The scanning strategy differs fundamentally from the B-Machine model (e.g. different elevation, frame, or rotation rate) and the fixed 45-degree/70-second parameters would be invalid

## Inputs

- Instrument scanning model parameters (elevation, rotation period, reference frame — e.g. 45 deg elevation, 70 s revolution, Equatorial frame)
- HEALPix Nside (512 for the B-Machine simulation)
- Sampling rate (33.4 Hz for B-Machine)
- Simulation duration or number of samples (~2000 samples per 60-second baseline interval)

## Outputs

- Pointing timeline with TIME, PHI, THETA, PSI, FLAG columns serialized to a single HDF5 file in the dst_io.read_data dtype
- Sky coverage and hit-count statistics (e.g. 44.8% coverage, median 20 hits/pixel at Nside 512)

## How to apply

Define the instrument's scanning model — for the B-Machine prototype this is fixed elevation scanning at 45 degrees with 1 revolution every 70 seconds in the Equatorial reference frame. Use HEALPix pixelization at Nside 512 to associate pointing angles with sky pixels, yielding about 44.8% sky coverage and a median hit-count of 20 hits per pixel. Sample the pointing at the polarimeter's acquisition rate (33.4 Hz for B-Machine) and serialize the resulting PHI, THETA, PSI angles together with TIME and FLAG into the single-HDF5 format expected by dst_io.read_data (dtype includes >f8 TIME/PHI/THETA/PSI, >i2 FLAG, >f4 TEMP/Q/U). The rationale is that destriping requires a scanning strategy with frequent crossing points, so the pointing model must reproduce the real instrument's sky coverage before a signal-only or signal-plus-noise timestream can be generated.

## Related tools

- **HEALPix** (Pixelizes the sky and defines the PHI/THETA pointing grid at Nside 512 used for the scanning timeline)
- **dst** (Consumes the generated pointing columns via dst_io.read_data as part of the destriping input HDF5 file) — http://github.com/zonca/dst
- **Python** (Implementation language for computing the pointing timeline and writing the HDF5 output)
- **B-Machine** (Instrument whose scanning geometry (45 deg elevation, 70 s revolution, 33.4 Hz sampling) defines the pointing model parameters)

## Examples

```
python generate_pointing.py --elevation 45 --rev-period 70 --frame equatorial --nside 512 --sample-rate 33.4 --output pointing.h5
```

## Evaluation signals

- Output HDF5 file contains TIME, PHI, THETA, PSI, FLAG columns with the expected dtype (e.g. >f8 for angles, >i2 for FLAG)
- Timeline length matches the expected sample count (~2000 samples per 60-second baseline interval at 33.4 Hz)
- Sky coverage computed from the pointing is about 44.8% at Nside 512
- Median hit-count per pixel is about 20 at Nside 512
- The file is a valid single HDF5 file readable by dst_io.read_data without modification to the reader

## Limitations

- The pointing model is specific to B-Machine's scanning geometry (45 deg elevation, 70 s revolution); other instruments require re-deriving the model
- Fixed ~44.8% sky coverage with median 20 hits/pixel means the simulation does not represent full-sky surveys; destriping requires frequent crossing points, so sparse strategies may degrade performance
- The dst reader only supports a single HDF5 file with a fixed column dtype; other formats require modifying dst_io.read_data

## Evidence

- [methods] Fixed elevation scanning at 45 degrees, 1 revolution each 70 seconds, Equatorial frame, ~44.8% sky coverage, median hit-count 20 per pixel at Nside 512: [evidence span withheld — anchor status: not_found]
