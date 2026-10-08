[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.nm000355-blue)](https://doi.org/10.82901/nemar.nm000355)

# Stolk et al. 2018 FieldTrip iEEG protocol dataset (SubjectUCI29): the authors' preprocessed epochs

## Overview

**These are not raw recordings.** This dataset is a BIDS packaging of the *preprocessed* intracranial EEG released
with the FieldTrip human intracranial analysis protocol (Stolk et al., 2018, Nature Protocols). The authors did not share the
raw recording: "Raw recording files are not shared, in order to protect the subject's identity." What they released, and
what is packaged here, is the output of protocol steps 35-36: 26 trials of 152 channels at 5000 Hz, cut from -0.4 to
0.9002 s around tone onset, demeaned, low-pass filtered at 200 Hz and band-stop filtered at 59-61, 119-121 and
179-181 Hz. `dataset_description.json` therefore declares `DatasetType: derivative`, with `GeneratedBy` and
`SourceDatasets` pointing at the authors' Zenodo record (doi:10.5281/zenodo.1201560, v1.0, CC-BY-SA-4.0).

## Cohort and recording

- One adult patient with medication-refractory epilepsy (`sub-UCI29`, source ID `SubjectUCI29`), recorded at the
  University of California, Irvine Medical Center. Study approved by the Office for the Protection of Human Subjects of
  the University of California, Berkeley; the subject gave informed consent (as stated by the authors). Age, sex and
  handedness are not given in the release.
- Electrodes: 56 SEEG contacts on 7 depth leads (RAM/LAM amygdala, RHH/LHH hippocampal head, RTH/LTH hippocampal tail,
  ROC right occipital) and 96 ECoG contacts (LPG 64-contact left parietal grid, LTG 32-contact left temporal grid).
  `SubjectUCI29_grids.png` in `sourcedata/` is the authors' schematic of the grid numbering.
- Task: the patient pressed a button with the right hand on hearing a target tone. Trials are aligned to tone onset
  (trigger value 4).

Additional acquisition details (Stolk et al. 2018, "Materials" and "Experimental design"; PMC6548463):

- Implant: grids LPG (64 contacts, 8 x 8, left parietal) and LTG (32 contacts, 4 x 8, left temporal), Integra,
  10 mm inter-electrode spacing; depth leads LAM, LHH, LTH, RAM, RHH, RTH and ROC, 8 contacts each, Ad-Tech, 5 mm
  inter-electrode spacing. Implanted "as part of the preparation for the epilepsy surgery".
- Amplifier: "All neural recordings were acquired using a Nihon Kohden recording system with a JE-120A amplifier
  (Nihon Kohden Corporation, Tokyo, Japan), analogfiltered above 0.01 Hz, and digitally sampled at 5 KHz".
- Reference: the acquisition reference electrode is not stated for this recording (see `iEEGReference`).
- Anatomical imaging used by the authors (not redistributed here): pre-implant T1 MRI (Siemens 3T TrioTim),
  post-implant CT (Philips iCT 256), post-implant T1 MRI (Siemens 1.5T Avanto).

## Task / paradigm and timing

"The neural data were recorded in the context of an experiment that required the patient to press a button with
the right hand when hearing a target tone" (Stolk et al. 2018). The authors defined trials from trigger value 4
(tone onset) in the recording's trigger channel, from 400 ms before to 900 ms after the tone (protocol step 34:
`cfg.trialdef.eventvalue = 4; prestim = 0.4; poststim = 0.9`), giving the "experiment's twenty-six trials" (step 36).
Button-press times are not in the release as events; the meaning of `trialinfo` is not documented (see
`_events.json`).

## Files

- `sub-UCI29/ieeg/sub-UCI29_task-tonedetection_ieeg.vhdr/.vmrk/.eeg`: BrainVision, 152 channels, 5000 Hz, IEEE float32.
  The 26 epochs (6502 samples each) are stored **back to back**. The sidecar says `RecordingType: epoched` and
  `EpochLength: 1.3004`, and the marker file has a `New Segment` at each epoch start. The file's time axis is
  not the original recording's time axis. Do not filter across epoch boundaries as if the data were continuous.
  To get epochs, split the file every 6502 samples. Inside each epoch, sample 2000 (0-based) is tone onset (t = 0).
  `mne_bids.read_raw_bids` refuses epoched recordings. You can read the file with `mne.io.read_raw_brainvision` and
  then reshape it.
- `..._events.tsv`: one `epoch` row and one `tone` row per trial. The rows also carry the authors' `data.trialinfo`
  value, which the release does not explain, and `data.sampleinfo`, the epoch's begin and end sample in the original
  unshared recording. Use `sampleinfo` to recover the original spacing between trials.
- `..._channels.tsv`: channel type (SEEG/ECOG), lead group and filter settings. **Units:** the FieldTrip structure has
  no unit field. The stored amplitudes (median |x| ≈ 49, max |x| ≈ 1398) are only plausible as microvolts, so µV is
  used here. The numbers are the authors' values.
- `sub-UCI29_space-ACPC_electrodes.tsv` + `_coordsystem.json`: x/y/z are `elec_acpc_fr`, the subject-ACPC positions
  the authors attached to the data (CT-MRI fusion, then brain-shift compensation of the grids). Further columns give
  `elec_acpc_f` (before brain-shift compensation), `elec_mni_frv` (volume-based MNI normalisation; the template
  variant is not stated by the authors) and `elec_fsavg_frs` (fsaverage, grids only). All are in mm, as stored.
- `sourcedata/zenodo-1201560/`: the authors' non-imaging files, byte-identical, with SHA-256 in
  `sourcedata/sourcedata_provenance.json`. These are the FieldTrip data (`_data.mat`, float64), header, the four
  electrode structures, the electrode table, the time-frequency result `_freq.mat` (a further derivative computed after
  re-montage) and the left cortical hull mesh.

## Preprocessing already applied by the source

Protocol step 35 (Stolk et al. 2018), applied by the authors before release with FieldTrip `ft_preprocessing`:
`cfg.demean = 'yes'; cfg.baselinewindow = 'all'; cfg.lpfilter = 'yes'; cfg.lpfreq = 200; cfg.padding = 2;
cfg.padtype = 'data'; cfg.bsfilter = 'yes'; cfg.bsfiltord = 3; cfg.bsfreq = [59 61; 119 121; 179 181]`.
The later protocol steps (bad-segment rejection, re-montage to common average for grids and bipolar for depths,
time-frequency analysis) were not applied to the packaged signals; `SubjectUCI29_freq.mat` in `sourcedata/` is the
authors' time-frequency result from those later steps. Hardware filtering: analog high-pass above 0.01 Hz
(`low_cutoff` in `_channels.tsv`).

## Known caveats

- Not raw data: authors' preprocessed epochs only (see Overview).
- Epochs are stored back to back; the file time axis is not the recording time axis (see Files).
- Units (µV) are inferred from amplitudes, not stated by the source (see Files).
- `trialinfo` is undocumented by the authors.
- Head imaging is not redistributed (see "What is not included, and why").

### Precision

The source stores float64 and BrainVision float32. The largest absolute difference between the BrainVision samples and
the source values is reported in the conversion summary. It is below 1e-4 µV, against a signal of tens of µV. For
bit-exact values, use `sourcedata/zenodo-1201560/SubjectUCI29_data.mat`.

### What is not included, and why

The record's head imaging is not redistributed here. That covers the pre-implant MRI, the post-implant MRI, the CT, the
CT/MRI overlay figure and `freesurfer.zip`, whose subject directory contains whole-head T1/orig/rawavg volumes. The
authors state the imaging was defaced with `ft_defacevolume`, and our low-resolution renders show rectangular defacing
masks. We did not run an independent full-resolution identifiability review. These files remain available from the
authors at https://zenodo.org/records/1201560. The same data are also archived at the Donders Repository
(hdl:11633/di.dccn.DSC_3015000.00_734; Zenodo marks it as identical).

## How to load

`mne_bids.read_raw_bids` refuses epoched recordings, so read the BrainVision file directly and cut it into the
26 epochs of 6502 samples (tone onset at sample 2000 of each epoch):

```python
import mne
import numpy as np

raw = mne.io.read_raw_brainvision(
    "sub-UCI29/ieeg/sub-UCI29_task-tonedetection_ieeg.vhdr", preload=True)
data = raw.get_data()                      # (152, 26 * 6502)
epochs = data.reshape(152, 26, 6502).transpose(1, 0, 2)
times = (np.arange(6502) - 2000) / raw.info["sfreq"]   # -0.4 ... 0.9002 s
```

The signal files are stored with git-annex on NEMAR; fetch them first (for example `git annex get sub-UCI29`).
Electrode positions are in `sub-UCI29_space-ACPC_electrodes.tsv` (subject ACPC, mm).

## Licence and citation

CC-BY-SA-4.0 (from the Zenodo record), so adaptations must be shared under the same licence. Please cite Stolk et al.
(2018), doi:10.1038/s41596-018-0009-6, and the data record doi:10.5281/zenodo.1201560.

## Source and provenance

- Data record: Zenodo doi:10.5281/zenodo.1201560 (v1.0, published 2018-03-21; identical copy at the Donders
  Repository, hdl:11633/di.dccn.DSC_3015000.00_734).
- Article: Stolk et al. (2018) Nature Protocols 13:1699-1723, doi:10.1038/s41596-018-0009-6 (author manuscript
  PMC6548463; preprint doi:10.1101/230912). Acquisition, implant and funding details above were read from the PMC
  full text on 2026-10-06.
- Packaging: `laneD_convert.py` (iEEG-NEMAR campaign); see `GeneratedBy` in `dataset_description.json` and
  SHA-256 checksums in `sourcedata/sourcedata_provenance.json`.
