---
name: baseline-output-file-verification
description: Use when you have just installed galkin and run galkin_data.py under
  the baseline (default) Galactic parameter setup, and need to check that the generated
  rotation-curve, positional and raw data files match the reference files shipped
  with the distribution.
when_to_use_negative:
- You ran galkin_data.py with non-default parameters (e.g. R0=8.5 kpc or R0=7.5 kpc
  variants) — the baseline reference files only apply to the default parameter set.
- 'You intend to use the raw data files directly: the README warns not to use galkin/data/
  files or bin/output/rawdata.dat unless fully aware of each original reference''s
  assumptions, nomenclature and Galactocentric parameters.'
- Your goal is repeated on-the-fly parameter scanning — use galkin_data_fast.py instead,
  which avoids input/output files and thus has no baseline file comparison path.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: galkin
  role: Python package providing the data handling routines that galkin_data.py invokes
    to normalise and output the rotation curve data
  repo: github.com/galkintool/galkin
- name: galkin_data.py
  role: Data retriever script whose output files are the subject of the baseline verification
  repo: github.com/galkintool/galkin
- name: Python 2.7.6
  role: Runtime environment required to install and run the galkin scripts (2.7.6
    or higher)
- name: numpy
  role: Required python package dependency of the galkin code
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:4b3854caa03ad308e8315e947840d6831b9d135d9e15b79aff43beb0d639ba94
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python 2.7.6
  - galkin
  - galkin_data.py
  - numpy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# baseline-output-file-verification

## Summary

Verify that a fresh run of galkin_data.py reproduces the deposited baseline outputs (vcdata_baseline.dat, posdata_baseline.dat, rawdata.dat) for the default Galactic parameter set, confirming the tool compiled the 2780 gas, star and maser measurements correctly.

## When to use

Use when you have just installed galkin and run galkin_data.py under the baseline (default) Galactic parameter setup, and need to check that the generated rotation-curve, positional and raw data files match the reference files shipped with the distribution.

## When NOT to use

- You ran galkin_data.py with non-default parameters (e.g. R0=8.5 kpc or R0=7.5 kpc variants) — the baseline reference files only apply to the default parameter set.
- You intend to use the raw data files directly: the README warns not to use galkin/data/ files or bin/output/rawdata.dat unless fully aware of each original reference's assumptions, nomenclature and Galactocentric parameters.
- Your goal is repeated on-the-fly parameter scanning — use galkin_data_fast.py instead, which avoids input/output files and thus has no baseline file comparison path.

## Inputs

- galkin_data.py run in input-file mode with inputpars.txt (baseline parameter specifications)
- Original kinematic data files galkin/data/data_*.dat (Table 1 source references)
- Deposited reference outputs output/vcdata_baseline.dat and output/posdata_baseline.dat

## Outputs

- output/vcdata.dat (normalised rotation curve) to be compared with vcdata_baseline.dat
- output/posdata.dat (positional data) to be compared with posdata_baseline.dat
- output/rawdata.dat (un-normalised original-reference data)

## How to apply

After unpacking the tar file, install with 'python setup.py install' under Python 2.7.6 (or higher), then run the data retriever in input-file mode: 'python galkin_data.py inputpars.txt' using the provided sample parameter file. The script processes the original kinematic data under galkin/data/data_*.dat (sourced from Table 1 of the manuscript) and writes output/vcdata.dat, output/posdata.dat and output/rawdata.dat. For the baseline Galactic parameter setup (default), diff or otherwise compare the generated vcdata.dat and posdata.dat against the deposited reference files output/vcdata_baseline.dat and output/posdata_baseline.dat, which the documentation prescribes 'for checking purposes'. Note that rawdata.dat is given as in the original references and is NOT normalised to the selected input parameters, so it should only be compared for raw provenance, not as a normalised-output check. A successful verification is an exact match of the regenerated files with their baseline counterparts, consistent with the compilation of 2780 gas, star and maser measurements optimised for Galactocentric radii between 3 and 20 kpc.

## Related tools

- **galkin** (Python package providing the data handling routines that galkin_data.py invokes to normalise and output the rotation curve data) — github.com/galkintool/galkin
- **galkin_data.py** (Data retriever script whose output files are the subject of the baseline verification) — github.com/galkintool/galkin
- **Python 2.7.6** (Runtime environment required to install and run the galkin scripts (2.7.6 or higher))
- **numpy** (Required python package dependency of the galkin code)

## Examples

```
python galkin_data.py inputpars.txt && diff output/vcdata.dat output/vcdata_baseline.dat && diff output/posdata.dat output/posdata_baseline.dat
```

## Evaluation signals

- Regenerated output/vcdata.dat matches output/vcdata_baseline.dat exactly (file-level diff is empty).
- Regenerated output/posdata.dat matches output/posdata_baseline.dat exactly.
- Regenerated output/rawdata.dat reflects the un-normalized original references, i.e. it is NOT identical to the normalised vcdata/posdata outputs.
- The compiled dataset totals 2780 gas, star and maser measurements covering Galactocentric radii between 3 and 20 kpc.
- The run was executed under Python 2.7.6 or higher with matplotlib, numpy (and wx only for window mode) installed.

## Limitations

- Baseline reference files apply only to the baseline (default) Galactic parameter setup; other parameter choices require the R85/R75 sample outputs or no comparison at all.
- rawdata.dat contains data as in the original references and is not normalised to the selected input parameters; the user should only use these data if fully aware of all assumptions and nomenclature used in each original reference.
- The galkin/data/data_*.dat files contain the original published data of each reference with different Galactocentric parameters and definitions, often overlapping in observed sources, and must not be used directly.
- The code runs under python 2.7.6 (or higher) and requires matplotlib, numpy, wx (window mode only) and astropy (only for astropy-based coordinate conversion).

## Evidence

- [readme] For the baseline Galactic parameter setup (default), these files can be compared to the files output/vcdata_baseline.dat and output/posdata_baseline.dat for checking purposes.: "For the baseline Galactic parameter setup (default), these files can be compared to the files output/vcdata_baseline.dat and output/posdata_baseline.dat for checking purposes."
- [readme] Notice however that these raw data are given as in the original reference and thus not normalised to the selected input parameters.: "Notice however that these raw data are given as in the original reference and thus not normalised to the selected input parameters."
- [methods] After unpacking the tar file, install by typing 'python setup.py install': "After unpacking the tar file, install by typing 'python setup.py install'"
- [methods] The code outputs rotation curve and positional data in the files output/vcdata.dat and output/posdata.dat, respectively: "The code outputs rotation curve and positional data in the files output/vcdata.dat and output/posdata.dat, respectively"
- [abstract] optimised to Galactocentric radii between 3 and 20 kpc and includes the kinematics of gas, stars and masers in a total of 2780 measurements: "optimised to Galactocentric radii between 3 and 20 kpc and includes the kinematics of gas, stars and masers in a total of 2780 measurements"
