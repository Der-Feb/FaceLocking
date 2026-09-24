# FaceLocking

Face recognition that locks onto one enrolled identity and displays that
person's expression (`SMILE` or `NEUTRAL`) and eye state (`EYES_OPEN` or
`EYES_CLOSED`). Other faces never provide signals for the target.

## Setup

Install system packages, then use Python 3.11 to create a virtual environment
and install dependencies:

```bash
sudo apt update
sudo apt install -y python3.11 python3.11-venv python3-pip
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

If `python3.11` is not available in your Kali repositories, install it with
`pyenv` or use the default `python3` (check with `python3 --version`) and
adjust the `venv` command accordingly.

Place a compatible ArcFace ONNX model at either:

```text
models/embedder_arcface.onnx
models/w600k_r50.onnx
```

Model weights and face databases are deliberately ignored by Git.

## Enroll identities

```bash
python -m src.enroll --name="Enrolled_name"
```

Press `SPACE` to take samples, `a` for automatic capture, `s` to save after
the requested sample count, or `q` to quit. Enrollment is stored locally at
`data/db/face_db.npz`.

To keep your existing enrollments when moving from the original project,
copy these local files yourself (do not commit them):

```bash
cp old-project/data/db/face_db.npz FaceLocking/data/db/face_db.npz
cp old-project/data/db/face_db.json FaceLocking/data/db/face_db.json
```

Create the destination folder first if it does not exist:

```bash
mkdir -p FaceLocking/data/db
```

## Track one person

```bash
python -m src.face_tracking --name="Enrolled_name"
```

`--name` must be an exact enrolled name. The tracker shows `NOT_DETECTED`
when that person leaves the frame and does not switch to another face. Press
`Q` to quit.

## Webcam access

If the camera cannot be opened, make sure your user can access video devices:

```bash
ls /dev/video*
sudo usermod -aG video $USER
```

Log out and back in for the group change to take effect. If you are running
Kali in a virtual machine, attach the webcam to the VM from your hypervisor's
USB/device menu.

## Tuning signals

`src/face_signals.py` contains the eye and smile thresholds. Smile tolerance
is adjusted slightly for smaller faces to account for noisier distant
landmarks. `--stable-frames` controls how many matching frames are required
before an expression or eye state display changes.