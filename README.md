[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on004661-blue)](https://doi.org/10.82901/nemar.on004661)

Participants (N=17, all males) with an average age of 32.8 years performed a guided visual search task in parallel with a second binaurally presented auditory task (Ries, et al., 2016). EEG data from each participant were recorded using a 64-channel BioSemi ActiveTwo system digitized at 512 Hz. Four external electrodes were used to record bipolar horizontal and vertical EOG signals, and a single external electrode was placed on each of the left and right mastoids to provide the reference signals. Fourteen participants were included in the original study, with three additional participants later added, resulting in 17 participants.

The visual search task for this experiment required participants to follow a red annulus around the screen and press a button if the annulus stopped at a prespecified target. The auditory task for this experiment was an N-back matching task in which participants listened to a string of numbers presented at approximately 2 second intervals and were required to indicate whether the current number matched a previously presented number. For the N=0, this would be the number immediately prior. For N=1 this would be the number one level before that, and so on. In the example string “1”, “1”, “2”, “1”, “3”, “2”, the second “1” should generate a match in the N=0 condition, the third “1” should generate a match in the N=1 condition, and the second “2” should generate a match in the N=2 condition. The task was composed of a baseline condition in which participants were presented with both visual and auditory stimuli but were instructed to ignore the auditory component. Next, were three dual-task conditions with N-back levels of N=0, N=1, and N=2. 

## NEMAR curation changes (2026-05-21)

BIDS validator: 17 errors + 767 warnings → 0 errors + 510 warnings. Raw `.set`/`.fdt` binary payloads unchanged.

### `dataset_description.json`
- Added `DatasetType: "raw"`. Why: BIDS-validator otherwise infers a derivative-rules cascade when `DatasetType` is missing alongside `GeneratedBy`, producing spurious warnings.
- Added `GeneratedBy: [{Name: "nemar-cli", Version: "0.8.8", CodeURL: "https://github.com/nemar-org/nemar-cli"}]`. Why: records the NEMAR rehost step in the dataset's provenance chain.
- `ReferencesAndLinks`: `[""]` → `[]`. Why: an empty-string element is not a valid reference URL.

### `task-nback_events.json`
- Rewrote the sidecar to describe the columns that actually appear in the per-recording `_events.tsv` files: `onset` (with `Units: "s"`), `duration`, `sample`, and `value`. Why: the prior version declared a single `value` column with a 107-entry `Levels`/`HED` block whose keys (`1`, `1102`, `1103`, `1110`, ..., a richer scheme from an earlier acquisition or sibling dataset) did not match any of the cells actually present in this rehosted version. Every events.tsv here uses only two trigger codes — `10` and `11` — and neither appears as a key in the old Levels, so the validator was rejecting every cell as the wrong type. Closes all 17 `TSV_VALUE_INCORRECT_TYPE:value` errors.
- The old `HED` dict has been dropped along with the orphan `Levels`. Why: keeping a HED annotation block whose keys never match the data would be misleading; the `value` column is now declared as free-form numeric and the actual observed codes are listed in the column's `Description`. (If a future curator recovers the correct mapping from the original lab, the HED block can be re-added with keys `10` and `11`.)

### `task-nback_eeg.json` (new, inheriting root sidecar)
- Created at the dataset root carrying fields common to every recording: `EEGPlacementScheme: "10-10"` (channel names Fp1/AF7/AF3/F1/F3/F5/F7/FT7/FC5/FC3/FC1/C1/.../O1 are 10-10 labels), `MISCChannelCount: 0` and `TriggerChannelCount: 0` (every `channels.tsv` has 64 EEG rows after the type/units fix below, and zero rows of any other type), `Manufacturer: "BioSemi"` and `ManufacturersModelName: "ActiveTwo"` (stated explicitly in this README's "64-channel BioSemi ActiveTwo system" line), and `TaskDescription` (paraphrased verbatim from this README). Why: each of these closes one warning per recording (17 × 6 fields = 102 warning instances) via BIDS inheritance without modifying any of the 17 per-recording `_eeg.json` files.

### `sub-*/eeg/sub-*_task-nback_channels.tsv` (17 files)
- `type` column: `n/a` → `EEG` for all 64 rows in each file. Why: the validator's `EEG_CHANNEL_COUNT_MISMATCH` rule compares the count of `type=EEG` rows against the `EEGChannelCount` declared in `_eeg.json` (`64`); leaving the column at `n/a` made the validator see zero EEG rows.
- `units` column: `n/a` → `uV` for all 64 rows. Why: the README documents BioSemi ActiveTwo at 64 channels, which records microvolt-scale scalp EEG; declaring the unit avoids a unit-missing warning per row.
