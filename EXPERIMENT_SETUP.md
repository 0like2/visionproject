# Shot Capture Experiment

This prototype is meant to run from the `cv` conda environment.

```bash
conda activate cv
```

If another teammate needs the same environment from scratch:

```bash
conda env create -f visionproject/environment.yml
conda activate cv
```

On macOS, `scrcpy` and `adb` can also be installed from the included Brewfile:

```bash
brew bundle --file visionproject/Brewfile
```

It supports several capture policies in one loop so you can compare them with the same scene.

Modes:

- `manual`: user decides when to capture
- `person1`: auto-capture when one person is detected in a stable and usable framing
- `count3`: auto-capture when exactly 3 people are detected
- `ratio`: auto-capture when total person bbox area ratio is inside a target range

## What "person1" means

`person1` is the new default mode.

It auto-captures only when all conditions hold for several frames:

- exactly 1 person is detected
- the person bbox area ratio is inside a target range
- the person center is close to the frame center
- the person bbox is not touching the frame borders

This is the closest simple baseline for:

"If one person is properly inside the frame, capture automatically."

## Recommended setup

### Android mirror

```bash
scrcpy --max-size=1080 --max-fps=30 --window-title HCI_phone
```

Then run the script on the mirrored phone region:

```bash
conda activate cv
python visionproject/shot_capture_experiment.py \
  --source screen \
  --screen-region 100,80,420,900 \
  --crop 20,120,380,680
```

`--screen-region` should cover the whole mirrored phone window.

`--crop` should cover only the camera preview area inside the phone UI. This is important because status bars and buttons can disturb the detector.

### Webcam test

```bash
conda activate cv
python visionproject/shot_capture_experiment.py --source webcam --webcam-index 0
```

## Key controls

- `1`: switch to manual mode
- `2`: switch to `person1` mode
- `3`: switch to `count3` mode
- `4`: switch to `ratio` mode
- `c` or `space`: capture immediately
- `q`: quit

## Useful tuning for person1

Default values:

- `--one-person-ratio-min 0.10`
- `--one-person-ratio-max 0.35`
- `--center-tolerance 0.18`
- `--border-margin-ratio 0.05`
- `--stable-frames 10`
- `--cooldown-sec 2.0`

Meaning:

- `one-person-ratio-min/max`: how large the detected person should appear
- `center-tolerance`: how far the bbox center may drift from the frame center
- `border-margin-ratio`: how much free space must remain from each border
- `stable-frames`: how long the condition must hold before auto-capture
- `cooldown-sec`: prevents repeated captures

If auto-capture is too strict:

- reduce `--stable-frames`
- increase `--center-tolerance`
- widen `--one-person-ratio-min/max`

If it captures too easily:

- increase `--stable-frames`
- reduce `--center-tolerance`
- increase `--border-margin-ratio`

## Output

Each run creates a new folder under `visionproject/captures_compare/<timestamp>/`.

Inside it:

- `manual/`
- `person1/`
- `count3/`
- `ratio/`
- `captures.csv`

Each capture stores:

- raw frame
- overlay frame with boxes and status text
- one CSV row with mode, trigger type, person count, area ratio, and elapsed time

## Suggested comparison flow

If you want a quick first experiment, compare these three:

1. `manual`
2. `person1`
3. `ratio`

That gives you:

- full user control
- simple single-subject auto capture
- composition-size based auto capture

## Important limitation

The current prototype uses OpenCV HOG person detection. It is easy to run in `cv` and does not need extra model files, but it can miss seated people or strong top-down views.

That is acceptable for a first comparison pass. If the loop and logging feel right, you can later replace only the detector with a stronger model while keeping the same experiment structure.
