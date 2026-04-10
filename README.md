# M-House Circular Variations

A MATLAB-based visual foraging experiment examining contextual learning and transfer. Participants search for targets hidden behind doors arranged in two concentric circles, learning context-specific target locations across three experimental stages.

---

## Overview

In each trial, participants are shown 20 doors arranged in two circles (8 inner, 12 outer) and must click doors to find a hidden target. The target is placed behind one of six specific doors with a fixed probability distribution. Across three stages the experiment measures how well participants learn, maintain, and transfer this spatial knowledge when the context changes.

**Three Experimental Stages**

| Stage | Name | Trials | Description |
|-------|------|--------|-------------|
| 1 | Learning | 200 per house | Participants learn two house layouts with structured target locations |
| 2 | Training | 160 | Contexts alternate at either a 5% or 30% switch rate (group-dependent) |
| 3 | Transfer Test | 120 | Performance tested on novel, permuted, and composite configurations |

**Progression Criterion**: participants must achieve ≥ 90% accuracy (find the target within 6 clicks) over the last 60 trials of each stage before advancing.

---

## Repository Structure

```
m-house_circular-variations/
├── README.md                          ← you are here
└── mforage_circular/
    ├── run_iforage_task.m             ← MAIN ENTRY POINT
    ├── generate_trial_structure_learn.m
    ├── generate_trial_structure_train.m
    ├── generate_trial_structure_lttest.m
    ├── door_setup.m
    ├── draw_*.m                       ← display / graphics helpers
    ├── query_*.m                      ← mouse input handlers
    ├── tally_moves.m
    ├── run_instructions*.m            ← instruction screens
    ├── take_a_break.m
    ├── test_trial_balancing.m
    ├── setup_sub_configs/             ← counterbalancing setup (run once)
    │   ├── generate_sub_configs.m
    │   ├── assign_target_locations.m
    │   ├── generate_legal_tasks.m
    │   └── check_legal.m
    ├── JSONio/                        ← JSON I/O library for MATLAB
    │   ├── jsonread.m / jsonwrite.m
    │   └── jsonread.mex*              ← pre-compiled MEX binaries
    ├── Assets/
    │   ├── tgts/                      ← target images (categories A–D)
    │   ├── Badges/                    ← Bronze / Silver / Gold / Champion PNGs
    │   ├── breakphotos/               ← break-screen images
    │   └── win/                       ← reward sound effects (MP3)
    └── exp_circ6/                     ← example subject data output
```

---

## Requirements

- **MATLAB** (R2018a or later recommended)
- **[PsychToolbox v3](http://psychtoolbox.org/)** — provides display, timing, audio, and input functions
- *(Optional)* SMI iViewX eye-tracker and driver software

---

## Setup

### 1 — Install PsychToolbox

Follow the [official PsychToolbox installation guide](http://psychtoolbox.org/download) for your platform (Windows / macOS / Linux). Ensure that `Screen`, `KbCheck`, `GetMouse`, and `PsychPortAudio` are all available from the MATLAB command window.

### 2 — Add the JSONio library to your MATLAB path

```matlab
addpath(genpath('mforage_circular/JSONio'))
```

Pre-compiled MEX binaries are provided for Windows (`.mexw64`), macOS (`.mexmaci64`), and Linux (`.mexa64`). If none of them work for your platform, recompile from source:

```matlab
cd mforage_circular/JSONio
mex -O jsonread.c
```

### 3 — Generate subject configurations (one-time, before data collection)

This step creates `sub_info.mat`, which stores counterbalancing information for up to 96 participants (48 per group × 2 groups).

```matlab
cd mforage_circular/setup_sub_configs
generate_sub_configs
```

### 4 — Configure the monitor

Open `mforage_circular/run_iforage_task.m` and update the monitor parameters near **line 234** to match your display:

```matlab
monitorXdim = 530;   % physical width in mm
monitorYdim = 300;   % physical height in mm
% viewing distance = 570 mm
```

---

## Running the Experiment

From the MATLAB command window, navigate to the `mforage_circular/` directory and call the main script:

```matlab
cd mforage_circular
run_iforage_task
```

The script will prompt for:

| Prompt | Values |
|--------|--------|
| Subject number | Any integer (must match a row in `sub_info.mat`) |
| Stage | `1` = Learning, `2` = Training, `3` = Transfer test |
| House number | `1` or `2` (only used in Stage 1) |
| Monitor location | `0` = office, `1` = lab |

### Output

Behavioral log files are saved to:

```
exp_circ6/sub-{N}/ses-{stage}/beh/sub-{N}-ses-{stage}_house-{H}_task-mforage_*.mat
```

Each file contains the full trial structure, target locations, and a row-by-row behavioral log with columns:

```
sub | stage | trial_num | context | timer | click_flag | door_idx |
door_prob | target_flag | 0 | x_coord | y_coord
```

---

## Experimental Design Details

### Circular door layout

- **Inner ring**: 8 doors at a radius of 30 mm
- **Outer ring**: 12 doors at a radius of 60 mm
- Context is indicated by background colour; each house has a distinct colour cue

### Stage 1 — Learning

- 200 trials per house (houses 1 and 2 run in separate sessions)
- 5 practice trials with random target placement
- Breaks every 20 trials with motivational badges

### Stage 2 — Training

- Two contexts alternate; switch probability depends on group assignment
  - **Group 1**: 5% switch rate
  - **Group 2**: 30% switch rate
- Maximum score: 28 000 points (250 per optimal trial)

### Stage 3 — Transfer Test

| Condition | Trials | Description |
|-----------|--------|-------------|
| Novel | 40 | Entirely new target-door configuration |
| Permuted | 40 | Same six target doors, different positional assignments |
| Composite | 40 | Combination of both previously learned contexts |

No performance feedback or rewards are given during the transfer test.

### Scoring and badges

Points per trial = max(7 - number\_of\_clicks\_to\_find\_target, 0).  
Motivational badges are awarded at cumulative point thresholds: **Bronze** → **Silver** → **Gold** → **Champion**.

---

## Counterbalancing

Subject configurations are generated by `setup_sub_configs/generate_sub_configs.m` and ensure:

- Each participant is assigned a unique combination of task sets (A, B, novel, permuted, composite)
- Target door configurations are not rotations or reflections of each other (validated by `check_legal.m`)
- Transfer order and colour assignments are systematically rotated across participants

---

## Tests

Two lightweight test scripts are included:

```matlab
% Validate trial-structure generation
cd mforage_circular
test_trial_balancing

% Validate JSON I/O
cd mforage_circular/JSONio
test_jsonread
test_jsonwrite
```

---

## Optional Eye-Tracking Integration

The repository includes `InitiViewXAPI.m` for optional integration with an SMI iViewX Red-m eye tracker. Initialise the API before calling `run_iforage_task`:

```matlab
InitiViewXAPI   % sets up the SMI iViewX connection
run_iforage_task
```

---

## BIDS Metadata

Helper scripts are provided for generating [Brain Imaging Data Structure (BIDS)](https://bids.neuroimaging.io/)-compliant metadata alongside behavioural outputs:

- `generate_meta_data_jsons.m` — top-level dataset metadata
- `generate_task_metadata.m` — task-specific JSON sidecar
- `generate_channel_loc_json.m` — electrode-location JSON (for EEG integration)

---

## License

See [LICENSE](LICENSE) for details (if present), or contact the repository owner.
