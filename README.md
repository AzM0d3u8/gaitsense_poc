# GaitSense PoC

Python-only feasibility pipeline: smartphone video → MediaPipe Pose Landmarker → landmarks → quality validation → smoothing → manually validated gait events → relative gait features → condition comparison → participant-grouped Random Forest baseline.

This is not a medical diagnostic system. `controlled_variation` means a participant was deliberately instructed to alter their gait; it does not mean disease or abnormality.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Download the MediaPipe Full Pose Landmarker model and place it at `models/pose_landmarker_full.task`. Put consented, locally stored videos in `videos/`; raw videos and model files are ignored by Git. Add participant IDs and conditions to `data/metadata.csv`.

## Required one-video validation

```bash
python scripts/01_video_info.py --video videos/P01_normal_01.mp4
python scripts/02_pose_extraction.py --video videos/P01_normal_01.mp4
python scripts/03_quality_check.py
python scripts/04_visualize_pose.py --video videos/P01_normal_01.mp4
```

Inspect the pose video and quality CSV. Do not proceed if the skeleton is unstable or quality is poor. Then run `05_smoothing.py`, `06_gait_events.py`, and `07_feature_extraction.py`; inspect event plots before comparison and ML. After the one-video inspection, `python run_pipeline.py` runs all stages.

Detected events are called `step_event`, not validated heel strikes. Coordinates are normalized image-relative values, not meters; the PoC does not report true speed or stride length. The ML split is by participant, never by video. Small samples have high uncertainty.

## Regression score prediction

`scripts/12_train_regression.py` provides Random Forest, Linear Regression, and scaled MLP regression baselines, split by participant. For walking-speed prediction, use the interactive measurement tool:

```bash
python scripts/measure_walking_speed.py --input-dir 'Test walking videos '
```

For each video, press `SPACE` on the first frame of the defined 4-meter trial, `ENTER` on the final frame, then `N` to save and continue. `R` resets; `Q` quits. The tool reads each video's actual FPS and calculates `4.0 / ((end_frame - start_frame) / fps)`. It writes reproducible measurements to `data/speed_measurements.csv` and targets to `data/speed_targets.csv`, preserving prior measurements unless `--overwrite` is used.

Then run:

```bash
python scripts/12_train_regression.py
```

The trainer reports missing labels, rejects identical labels, excludes `gait_pattern_index` and its components, and refuses insufficient data. `720p` means resolution; `60 FPS` means approximately 60 frames per second. Camera distance does not determine speed: the relevant distance is the participant's known 4-meter walking path. Do not use a pose-derived feature as its own target.

## Relative gait-pattern indicator

When no external score exists, run `python scripts/13_compute_pattern_index.py`. This creates `data/features_with_pattern_index.csv` with a transparent 0–100-ish relative indicator based on percentile ranks of timing asymmetry, knee asymmetry, arm-swing asymmetry, and step-time variability. It is suitable only for research exploration or deciding whether to repeat/review a recording. It is not a clinical score, diagnosis, disease prediction, or medical precautionary alert, and it must not be used as the target for a model trained on those same features.

## Reference gait-parameter sample

The checked-in `dataset_samples/gait_parameters.csv` is treated as the reference parameter table, while `gait_parameters_estimation.csv` is treated as pose-derived estimates. Prepare a comparison with `python scripts/15_prepare_reference_gait_parameters.py`. The reference `Velocity_UGS`/`Velocity_FGS` columns are possible independent speed targets for a larger dataset. The included sample has only one row (`PA000`), so it cannot train or evaluate ML; it is useful for confirming the schema and label relationship.
